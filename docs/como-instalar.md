# Instalação [Login Cidadão](https://github.com/PROCERGS/login-cidadao)
Esta documentação foi baseada em um servidor GNU/Linux Debian 8.1. Se sua intenção é instalar em sistema de base diferente algumas adaptações podem ser necessárias. 
## Server User
Para instalar a aplicação é necessário ter acesso a dois usuários: um usuário padrão e um usuário com poderes de sudo. Você pode usar o usuário padrão de costume de seu servidor e o root, mas recomendamos criar um usuário só para o gerenciamente do Login Cidadão, facilitando assim o controle e o registro de logs do sistema. 
```
    // logado como root, crie o novo user:
    # useradd --create-home --groups sudo -s /bin/bash login-cidadao
    // Insira uma senha para o novo user
    # passwd login-cidadao
``` 

## Instalando Dependências
Para que o Login Cidadão funcione corretamente será necessário que estejam instaladas as seguintes dependências: 
* PHP >=5.3.3
* composer
* node.js
* PHP Extensions
  * php5-curl
  * php5-intl
  * php5-mysql ou php5-pgsql ou integração de base de dados de sua preferência
  * php5-memcache (Também é possível usar php5-memcached mas será necessário mudar algumas classes de Memcache para Memcached)

```
    // Instalando pacotes do php. 
    // Observe que aqui vamos optar por usar o postgres, mas é possível usar mysql sem problemas
    $ sudo apt-get install php5 php5-curl php5-intl php5-pgsql php5-memcache
    
    // Instalando Composer
    $ sudo apt-get install curl
    $ sudo curl -sS https://getcomposer.org/installer | php
    $ sudo mv composer.phar /usr/local/bin/composer
    
    // Instalando nodejs
    $ sudo curl -sL https://deb.nodesource.com/setup_5.x | bash -
    $ sudo apt-get install --yes nodejs
```

#### [Obtendo o Login Cidadão e instalando](id:obter)
 
Após instalar as dependências, clone o repositório da aplicação. Recomendamos fazer isso com o usuário criado e dentro de uma estrutura de diretórios padrão /var/www/login-cidadao, mas é possível adaptar para qualquer contexto. 
```
    $ `git clone https://github.com/PROCERGS/login-cidadao.git`
```
 
2. Entre no diretório criado  
    `cd login-cidadao`
 
3. Mude para o branch `symfony2.7` (este branch é experimental atualmente, 07.07.2015, mas funciona)  
    `git checkout symfony2.7`
 
4. Crie um arquivo `app/config/parameters.yml` a partir de `app/config/parameters.yml.dist` e edite com suas modificações
 
5. Verifique se todas os requisitos estão sendo cumpridos antes de iniciar a instalação
    `php app/check.php`
 
6. Se a verificação for bem sucedida inicie a instalação
    `./install.sh`
 
#### Arquivo de configuração `parameters.yml`
 
`locale:` -> substitua pelo seu locale (ex. pt_BR)
 
`secret:` -> substitua por uma longa cadeia de letras, números e símbolos
 
`site_domain:` -> substitua pelo seu domínio/subdomínio
 
`recaptcha_public_key:` e `recaptcha_private_key:` -> gere essas chaves em https://www.google.com/recaptcha/
 
`registration.cpf.empty_time:` e `registration.email.unconfirmed_time:` -> define quanto tempo deve ser dado para que o usuário confirme o CPF e o email, respectivamente
 
`brute_force_threshold:` -> quantas tentativas devem ser toleradas antes de considerar um ataque de força bruta
 
 
#### Apache
 
Exemplo de arquivo de configuração do Apache.
 
```
<VirtualHost *:80>
    ServerName sub.dominio.com.br
    ServerAdmin usuario@email
 
    DocumentRoot /var/www/login-cidadao/web
 
    <Directory / >
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        allow from all
    </Directory>
 
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
 
* Em `DocumentRoot` é preciso apontar para o diretório `web`, neste exemplo o caminho completo é `/var/www/login-cidadao/web`.
 
* ServerName deve ser preenchido com o domínio (ex. dominio.com.br) ou subdomínio completo (ex. sub.dominio.com.br)
 
### [Primeiros passos pós-instalação](id:pos-instalacao)
 
1. Adicione os seguintes aliases ao seu arquivo `.bashrc`  
    `alias prod='php app/console --env=prod'`  
    `alias dev='php app/console --env=dev'`
 
2. Atualize o perfil do terminal  
    `source ~/.bashrc`
    * Obs.: Etapa desnecessária para logins futuros já que o .bashrc será executado no processo de login.
 
3. Processe e ative todos os assets  
    `prod assets:install`  
    `prod assetic:dump`
 
4. Dar poderes de super administrador para o primeiro usuário  
    `prod fos:user:promote <username> ROLE_SUPER_ADMIN`
 
    * Obs. 1: Substitua "<username>" pelo nome do usuário como mostrado na área superior direita da página, geralmente o que precede o '@' do email usado na hora da criação do usuário.
 
    * Obs. 2: Para confirmar visualmente o novo papel de super administrador faça um logout e depois um login. Junto ao nome deverá haver um campo 'impersonate', ver esse campo é a confirmação.
 
## Navegando em modo de desenvolvimento
 
Adicione `/app_dev.php` na URL.
 
## Alguns comandos práticos
 
Assume-se que as estapas 1 e 2 dos [Primeiros passos pós-instalação](#pos-instalacao) tenham sido cumpridos para seguir estes comandos.
 
* Limpar o cache  
    `prod cache:clear`  
    `dev cache:clear`  
    se não funcionar, em última instância use  
    `rm -rf app/cache/*`
* Criar ou atualizar os assets  
    `prod assets:install`  
    `prod assetic:dump`
* Criar ou atualizar os vendors (útil, por exemplo, quando se muda de branch)  
    `composer install`
 
## Adicionando serviços
 
em breve
 
## Integrando com o Mapas Culturais
 
```
'auth.provider' => 'OpauthLoginCidadao',
'auth.config' => array(
    'client_id' => 'minha_chave_publica',
    'client_secret' => 'minha_chave_privada',
    'auth_endpoint' => 'https://sub.dominio/oauth/v2/auth',
    'token_endpoint' => 'https://sub.dominio/oauth/v2/token',
    'user_info_endpoint' => 'https://sub.dominio/api/v1/person.json'
),
```
* Obs. 1: As chaves pública e privada são geradas na adição do serviço.  
* Obs. 2: substituir o domínio/subdomínio das três últimas linhas.
