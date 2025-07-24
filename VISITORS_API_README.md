# Visitors Portal API

API para gerenciamento do portal cativo de visitantes, responsável por autenticar e autorizar dispositivos na rede de visitantes do ambiente.

## 📋 Visão Geral

Esta API é responsável por:
- Coletar informações dos visitantes através de um formulário web
- Integrar-se ao controlador UniFi para autorizar dispositivos na rede de visitantes
- Gerenciar o acesso temporário à rede

## 🚀 Instalação

1. **Execute o script de instalação**:
   ```bash
   chmod +x install_api.sh
   sudo ./install_api.sh
   ```

2. **Verifique o status do serviço**:
   ```bash
   sudo systemctl status visitors-portal-api
   ```

## ⚙️ Configuração

### Arquivo de Configuração (`api_config.py`)

```python
# Configurações do UniFi Controller
UNIFI_CONFIG = {
    'host': '192.168.48.2',      # Endereço IP do controlador UniFi
    'port': 8443,                 # Porta da API do UniFi
    'username': 'portal.cativo',  # Usuário com permissões no UniFi
    'password': 'sua_senha',      # Senha do usuário
    'site': 'default',           # Site no UniFi
    'verify_ssl': False          # Verificar certificado SSL (True em produção)
}

# Configurações da API
API_CONFIG = {
    'host': '0.0.0.0',  # Endereço para escutar
    'port': 8082,       # Porta da API
    'debug': False      # Modo de depuração
}

# Configurações de Autorização para Visitantes
AUTH_CONFIG = {
    'default_duration': 1440,  # minutos (24 horas)
    'up_speed': 1024,         # Kbps de upload
    'down_speed': 1024,        # Kbps de download
    'data_limit': 1024         # MB de limite de dados
}
```

## 🔄 Gerenciamento do Serviço

- **Iniciar o serviço**:
  ```bash
  sudo systemctl start visitors-portal-api
  ```

- **Parar o serviço**:
  ```bash
  sudo systemctl stop visitors-portal-api
  ```

- **Reiniciar o serviço**:
  ```bash
  sudo systemctl restart visitors-portal-api
  ```

- **Verificar status**:
  ```bash
  sudo systemctl status visitors-portal-api
  ```

- **Visualizar logs**:
  ```bash
  sudo tail -f /var/log/portal_cativo/api.log
  ```

## 🌐 Endpoints da API

### `POST /api/authorize`

Autentica um visitante e autoriza seu dispositivo na rede.

**Parâmetros (JSON):**
```json
{
  "name": "Nome do Visitante",
  "email": "email@exemplo.com",
  "phone": "(00) 00000-0000",
  "accept_terms": true
}
```

**Resposta de Sucesso (200 OK):**
```json
{
  "success": true,
  "message": "Dispositivo autorizado com sucesso!",
  "redirect": "http://www.google.com",
  "duration_minutes": 1440,
  "up_speed": 1024,
  "down_speed": 1024,
  "data_limit_mb": 1024
}
```

## 🔒 Segurança

- A API deve ser acessível apenas pelo servidor web do portal cativo
- Recomenda-se o uso de HTTPS em produção
- Mantenha as credenciais do UniFi em segredo

## 🐛 Solução de Problemas

### Dispositivo não é autorizado
- Verifique se o endereço MAC está sendo corretamente obtido
- Confirme se as credenciais do UniFi estão corretas
- Verifique os logs em `/var/log/portal_cativo/api.log`

### Erros de Conexão
- Verifique se o controlador UniFi está acessível na rede
- Confirme se as portas necessárias estão abertas no firewall

---

Para mais informações, consulte a documentação completa do projeto.
