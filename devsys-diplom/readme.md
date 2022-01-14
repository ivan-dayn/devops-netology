# Курсовая работа по итогам модуля "DevOps и системное администрирование" - Дайн Иван


1. Создал виртуальную машину на Oracle VM VirtualBox, с помощью Vagrant, использованный конфигурационный файл:
```config
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.04"
  config.vm.provider :virtualbox do |vb|
  config.vm.network "forwarded_port", guest: 443, host: 443
  end
end
```
2. Установил ufw и разрешил к этой машине сессии на порты 22 и 443, при этом трафик на интерфейсе localhost (lo) должен ходить свободно на все порты.
```bash
sudo apt install ufw
sudo ufw allow ssh
sudo ufw allow https
sudo ufw allow in on lo
sudo ufw enable
```
3. Установил hashicorp vault.

Добавляю в систему GPG ключ HashiCorp
```bash
$ curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
```
Добавляю репозиторий HashiCorp Linux.
```bash
$ sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
```
Устанавливаю Vault, предварительно обновив установщик.
```bash
$ sudo apt-get update && sudo apt-get install vault
```
4. Cоздал центр сертификации по инструкции ([ссылка](https://learn.hashicorp.com/tutorials/vault/pki-engine?in=vault/secrets-management)) и выпустил сертификат для использования его в настройке веб-сервера nginx (срок жизни сертификата - 30 дней).
```bash
$ vault server -dev -dev-root-token-id root # запуск Vault в dev (ознакомительном) режиме с рутовым токеном
$ export VAULT_ADDR=http://127.0.0.1:8200   #задаю переменные окружения
$ export VAULT_TOKEN=root
```
Создаю корневой сертификат центра сертификации
```bash
$ vault secrets enable pki                                           # включаю pki в ветке pki
$ vault secrets tune -max-lease-ttl=87600h pki                       # срок жизни примерно 10 лет
$ vault write -field=certificate pki/root/generate/internal \        # генерирую корневой сертификат devsysdip.com
     common_name="devsysdip.com" \
     ttl=87600h > CA_cert.crt

$ vault write pki/config/urls \                                      # прописываю адреса CA и CRL
     issuing_certificates="$VAULT_ADDR/v1/pki/ca" \
     crl_distribution_points="$VAULT_ADDR/v1/pki/crl"
```
Создаю помежуточный CA
```bash
$ vault secrets enable -path=pki_int pki                             # включаю pki в ветке pki_int
$ vault secrets tune -max-lease-ttl=43800h pki_int                   # срок жизни примерно 10 лет
$ vault write -format=json pki_int/intermediate/generate/internal \  # генерирую в json формате, извлекаю CSR и записываю в файл pki_intermediate.csr
     common_name="devsysdip.com Intermediate Authority" \
     | jq -r '.data.csr' > pki_intermediate.csr
$ vault write -format=json pki/root/sign-intermediate csr=@pki_intermediate.csr \ 
     format=pem_bundle ttl="43800h" \                                 # подписываю промежуточный сертификат корневым 
     | jq -r '.data.certificate' > intermediate.cert.pem              # из json извлекаю сертификат в файл intermediate.cert.pem  
$ vault write pki_int/intermediate/set-signed certificate=@intermediate.cert.pem  # подписанный импортирую в Vault 
```
Назначение роли промежуточному центру
```bash
$ vault write pki_int/roles/devsysdip-dot-com \
     allowed_domains="devsysdip.com" \
     allow_subdomains=true \                            # роль для выдачи сертификатов для поддоменов devsysdip.com
     max_ttl="720h"
```
Создание сертификата для test.devsysdip.com
```bash
# записывю сертификат в json формате в файл test.devsysdip.com.crt
$ vault write -format=json pki_int/issue/devsysdip-dot-com common_name="test.devsysdip.com" ttl="720h" > test.devsysdip.com.crt

$ cat test.devsysdip.com.crt | jq -r .data.certificate > test.devsysdip.com.crt.pem   # в файл test.devsysdip.com.crt.pem записывю сертификат
$ cat test.devsysdip.com.crt | jq -r .data.issuing_ca >> test.devsysdip.com.crt.pem   # и дополняю его сертификатом от промежуточного центра для построения цепочки
$ cat test.devsysdip.com.crt | jq -r .data.private_key > test.devsysdip.com.crt.key   # записываю закрытый ключ
```
5. Установил корневой сертификат созданного центра сертификации в доверенные в хостовой системе.
Импортировал сертификат
![Импортировал сертификат](https://github.com/ivan-dayn/devops-netology/blob/main/devsys-diplom/dip-site.png)
6. Установите nginx.
```bash
$ sudo apt install nginx
$ sudo mkdir -p /var/www/devsysdip.com/html
$ sudo chmod -R 755 /var/www
$ sudo ln -s /etc/nginx/sites-available/devsysdip.com /etc/nginx/sites-enabled/

$ cat /etc/nginx/sites-available/devsysdip.com
server {
        listen 443 ssl;                         # Specify the listening port
        root /var/www/devsysdip.com/html;       # The path to the website files
        index index.html index.htm;             # Files to display if only the domain name is specified in the address
        server_name test.devsysdip.com;         # Domain name of this site
        ssl_certificate /etc/ssl/certs/test.devsysdip.com.crt.pem;
        ssl_certificate_key /etc/ssl/private/test.devsysdip.com.crt.key;
        location / {
                try_files $uri $uri/ =404;
                }
}
/etc/nginx/sites-enabled/devsysdip.com -> /etc/nginx/sites-available/devsysdip.com
```
7. По инструкции ([ссылка](https://nginx.org/en/docs/http/configuring_https_servers.html)) настройте nginx на https, используя ранее подготовленный сертификат:
  - можно использовать стандартную стартовую страницу nginx для демонстрации работы сервера;
  - можно использовать и другой html файл, сделанный вами;
8. Откройте в браузере на хосте https адрес страницы, которую обслуживает сервер nginx.
9. Создайте скрипт, который будет генерировать новый сертификат в vault:
  - генерируем новый сертификат так, чтобы не переписывать конфиг nginx;
  - перезапускаем nginx для применения нового сертификата.
10. Поместите скрипт в crontab, чтобы сертификат обновлялся какого-то числа каждого месяца в удобное для вас время.

## Результат

Результатом курсовой работы должны быть снимки экрана или текст:

- Процесс установки и настройки ufw
- Процесс установки и выпуска сертификата с помощью hashicorp vault
- Процесс установки и настройки сервера nginx
- Страница сервера nginx в браузере хоста не содержит предупреждений 
- Скрипт генерации нового сертификата работает (сертификат сервера ngnix должен быть "зеленым")
- Crontab работает (выберите число и время так, чтобы показать что crontab запускается и делает что надо)

## Как сдавать курсовую работу

Курсовую работу выполните в файле readme.md в github репозитории. В личном кабинете отправьте на проверку ссылку на .md-файл в вашем репозитории.

Также вы можете выполнить задание в [Google Docs](https://docs.google.com/document/u/0/?tgif=d) и отправить в личном кабинете на проверку ссылку на ваш документ.
Если необходимо прикрепить дополнительные ссылки, просто добавьте их в свой Google Docs.

Перед тем как выслать ссылку, убедитесь, что ее содержимое не является приватным (открыто на комментирование всем, у кого есть ссылка), иначе преподаватель не сможет проверить работу. 
Ссылка на инструкцию [Как предоставить доступ к файлам и папкам на Google Диске](https://support.google.com/docs/answer/2494822?hl=ru&co=GENIE.Platform%3DDesktop).
