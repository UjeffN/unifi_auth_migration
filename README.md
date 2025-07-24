# 🚀 Sistema de Autenticação e Portal Cativo para Redes Wi-Fi

## 📋 Visão Geral

Este projeto é composto por dois componentes principais que trabalham em conjunto para fornecer um sistema completo de autenticação e portal cativo para redes Wi-Fi, integrado ao UniFi Controller:

1. **Auth Project**: API Django para autenticação e gerenciamento de usuários
2. **Portal Cativo**: Interface web para autenticação de visitantes na rede Wi-Fi

## 🏗️ Arquitetura do Sistema

```
+---------------------+       +----------------------+       +------------------+
|                     |       |                      |       |                  |
|   Dispositivo do   |<----->|     Portal Cativo    |<----->|   Auth Project   |
|      Usuário       |  HTTP |  (Nginx + Python)    |  HTTP |    (Django)      |
|                     |       |                      |       |                  |
+---------------------+       +----------------------+       +------------------+
                                                                    |
                                                                    | API
                                                                    v
                                                         +------------------+
                                                         |                  |
                                                         |  UniFi Controller |
                                                         |                  |
                                                         +------------------+
```

## ✨ Funcionalidades Principais

### Auth Project
- 🔐 Autenticação de usuários via LDAP/Active Directory
- 📱 Gerenciamento de dispositivos por MAC address
- ⏱️ Controle de tempo de acesso à rede
- 📊 Dashboard administrativo
- 📈 Métricas e logs de acesso
- 🔄 Integração direta com UniFi Controller

### Portal Cativo
- 🌐 Interface responsiva para autenticação de visitantes
- 🔄 Redirecionamento automático para portal de autenticação
- 📝 Formulário de aceitação de termos de uso
- 🔗 Integração com a API de autenticação
- 📱 Design mobile-friendly

## 🛠️ Tecnologias Utilizadas

### Backend
- Python 3.8+
- Django 4.2+
- Django REST Framework
- UniFi API Client
- Gunicorn
- Nginx (como proxy reverso)

### Frontend
- HTML5
- CSS3
- JavaScript (Vanilla)
- Bootstrap 5
- Bootstrap Icons

### Banco de Dados
- SQLite (desenvolvimento)
- PostgreSQL (produção recomendada)

### Infraestrutura
- Nginx (servidor web e proxy reverso)
- Systemd (gerenciamento de serviços)
- Docker (opcional para implantação)

## 🚀 Guia de Instalação

### Pré-requisitos
- Ubuntu 20.04/22.04 LTS
- Python 3.8+
- Nginx
- UniFi Controller em execução

### 1. Clonando o Repositório

```bash
# Clonar o repositório
sudo mkdir -p /opt
cd /opt
sudo git clone [URL_DO_REPOSITORIO] .
```

### 2. Configuração do Auth Project

#### Instalação de Dependências
```bash
cd /opt/auth_project
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

#### Configuração do Ambiente
Crie um arquivo `.env` na raiz do projeto com as seguintes variáveis:

```bash
# Configurações do Django
DJANGO_SECRET_KEY=sua-chave-secreta-aqui
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1,seu-ip

# Configurações do UniFi
UNIFI_CONTROLLER_IP=192.168.48.2
UNIFI_CONTROLLER_PORT=8443
UNIFI_USERNAME=admin
UNIFI_PASSWORD=sua-senha
UNIFI_SITE_ID=default

# Configurações de E-mail (opcional)
EMAIL_HOST=smtp.example.com
EMAIL_PORT=587
EMAIL_HOST_USER=seu-email@example.com
EMAIL_HOST_PASSWORD=sua-senha
EMAIL_USE_TLS=True
```

#### Aplicando Migrações
```bash
python manage.py migrate
python manage.py createsuperuser
```

#### Iniciando o Servidor de Desenvolvimento
```bash
python manage.py runserver 0.0.0.0:8000
```

### 3. Configuração do Portal Cativo

#### Instalação de Dependências
```bash
cd /opt/portal_cativo
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

#### Configuração do Nginx
Crie um arquivo de configuração em `/etc/nginx/sites-available/portal-cativo`:

```nginx
server {
    listen 80;
    server_name seu-dominio.com.br;

    location / {
        proxy_pass http://localhost:8082;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /static/ {
        alias /opt/portal_cativo/static/;
    }
}
```

Habilite o site:
```bash
sudo ln -s /etc/nginx/sites-available/portal-cativo /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

#### Configuração do Serviço Systemd
Crie um arquivo de serviço em `/etc/systemd/system/visitors-portal-api.service`:

```ini
[Unit]
Description=Portal de Visitantes API
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/opt/portal_cativo
Environment="PATH=/opt/portal_cativo/venv/bin"
ExecStart=/opt/portal_cativo/venv/bin/python server.py
Restart=always

[Install]
WantedBy=multi-user.target
```

Habilite e inicie o serviço:
```bash
sudo systemctl daemon-reload
sudo systemctl enable visitors-portal-api
sudo systemctl start visitors-portal-api
```

### 4. Configuração do UniFi Controller

1. Acesse o UniFi Controller
2. Vá para Settings > Wireless Networks
3. Crie ou edite a rede Wi-Fi que usará o portal cativo
4. Habilite "Guest Policy"
5. Em "Guest Control", selecione "Redirect using hostname"
6. Adicione o hostname ou IP do seu portal cativo
7. Salve as configurações

## 🔒 Segurança

- Todas as senhas são armazenadas usando hashing seguro
- Conexões com o UniFi Controller usam HTTPS
- Aplicação configurada com cabeçalhos de segurança modernos
- Rate limiting implementado para evitar abusos
- Logs detalhados para auditoria de segurança

## 📊 Monitoramento

O sistema inclui:
- Logs detalhados em `/var/log/portal_cativo/`
- Métricas de desempenho
- Monitoramento de uso de largura de banda
- Alertas para atividades suspeitas

## 🔄 Manutenção

### Backups
```bash
# Backup do banco de dados
python manage.py dumpdata > backup_$(date +%Y%m%d).json

# Backup dos arquivos estáticos
tar -czvf static_backup_$(date +%Y%m%d).tar.gz /opt/portal_cativo/static/
```

### Atualizações
```bash
cd /opt/auth_project
source venv/bin/activate
git pull
pip install -r requirements.txt
python manage.py migrate
python manage.py collectstatic --noinput
systemctl restart gunicorn
```

## 🤝 Suporte

Para suporte, por favor abra uma issue no repositório do projeto ou entre em contato com a equipe de TI.

## 📄 Licença

Este projeto é licenciado sob a licença MIT - veja o arquivo [LICENSE](LICENSE) para detalhes.

---

Desenvolvido com ❤️ pela Equipe de TI da Câmara Municipal
