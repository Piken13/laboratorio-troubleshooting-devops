# 🔧 Guia Completo: Laboratório de Troubleshooting

> **Para iniciantes!** Este guia explica cada passo com contexto, para que você entenda **o que está fazendo e por quê**.

---

## 🧠 Entendendo o Desafio Primeiro

A aplicação tem **3 camadas** que precisam conversar entre si:

```
Usuário (navegador)
      ↓
  [Interface Web]  ← camada 1: o que você vê no browser
      ↓
    [API]          ← camada 2: a lógica/cérebro da aplicação
      ↓
[Banco de Dados]   ← camada 3: onde os dados ficam guardados
```

**A regra de ouro do troubleshooting:** Sempre comece pela camada mais externa (o que o usuário vê) e vá aprofundando até encontrar onde a cadeia quebrou.

---

## 🚀 Passo a Passo para Resolver

### FASE 1 — Acessar o Ambiente

Você recebeu uma **chave de acesso** (normalmente é um arquivo `.pem` ou uma senha SSH). Para conectar ao servidor Linux:

```bash
# Se recebeu um arquivo de chave (.pem):
ssh -i caminho/para/sua-chave.pem usuario@ip-do-servidor

# Se recebeu usuário e senha:
ssh usuario@ip-do-servidor
# (vai pedir a senha depois)
```

> 💡 **O que é SSH?** É como um "controle remoto" para servidores Linux. Você digita comandos no seu computador, mas eles executam no servidor remoto.

---

### FASE 2 — Visão Geral do Sistema (Diagnóstico Inicial)

Assim que entrar no servidor, rode esses comandos para ter uma foto do estado atual:

#### 2.1 — Ver quais serviços estão rodando
```bash
# Lista todos os serviços e seus status
systemctl list-units --type=service --state=running

# Versão mais simples para ver tudo de uma vez:
systemctl status
```

#### 2.2 — Ver se algum serviço falhou
```bash
# Mostra serviços com falha
systemctl --failed
```

#### 2.3 — Ver os processos ativos
```bash
# Lista processos rodando
ps aux

# Filtrar por nome (exemplo: nginx, apache, node, python)
ps aux | grep nginx
ps aux | grep python
ps aux | grep node
```

---

### FASE 3 — Investigar Cada Camada

#### 🌐 Camada 1: Interface Web (Nginx ou Apache)

```bash
# Ver status do servidor web mais comum (Nginx):
systemctl status nginx

# Se for Apache:
systemctl status apache2

# Ver os logs de erro do Nginx:
sudo tail -50 /var/log/nginx/error.log

# Ver logs de acesso:
sudo tail -50 /var/log/nginx/access.log
```

**O que procurar:**
- `active (running)` = está funcionando ✅
- `inactive (dead)` ou `failed` = está parado ❌

**Se estiver parado, reinicie:**
```bash
sudo systemctl start nginx
# ou
sudo systemctl restart nginx
```

---

#### ⚙️ Camada 2: API (pode ser Node.js, Python, Java, etc.)

```bash
# Ver processos na porta 3000 (porta comum de APIs Node.js):
sudo lsof -i :3000

# Ver processos na porta 8000 ou 8080 (Python/outros):
sudo lsof -i :8000
sudo lsof -i :8080

# Ver TODAS as portas em uso:
sudo ss -tlnp
# ou:
sudo netstat -tlnp
```

**Descobrir qual tecnologia é a API:**
```bash
# Listar arquivos de configuração e código:
ls /var/www/
ls /opt/
ls /home/
find / -name "*.py" -not -path "*/proc/*" 2>/dev/null | head -20
find / -name "package.json" -not -path "*/proc/*" 2>/dev/null | head -10
```

**Ver logs de aplicações comuns:**
```bash
# Logs do sistema (útil para qualquer serviço):
sudo journalctl -xe

# Logs de uma unit específica (substitua 'api' pelo nome real):
sudo journalctl -u api -n 100

# Últimas 100 linhas dos logs do sistema:
sudo tail -100 /var/log/syslog
```

---

#### 🗄️ Camada 3: Banco de Dados

Os bancos mais comuns em ambientes de lab são **MySQL**, **PostgreSQL** e **SQLite**.

```bash
# Verificar MySQL:
systemctl status mysql
# ou:
systemctl status mysqld

# Verificar PostgreSQL:
systemctl status postgresql

# Se estiver parado, iniciar:
sudo systemctl start mysql
sudo systemctl start postgresql
```

**Testar conexão com o banco:**
```bash
# MySQL:
mysql -u root -p
# (digitar a senha quando pedir)

# PostgreSQL:
sudo -u postgres psql
```

**Ver logs do banco:**
```bash
# MySQL:
sudo tail -50 /var/log/mysql/error.log

# PostgreSQL:
sudo tail -50 /var/log/postgresql/*.log
```

---

### FASE 4 — Verificar Conectividade Entre Camadas

Mesmo que todos os serviços estejam rodando, eles podem não estar se comunicando.

```bash
# Testar se a API responde localmente:
curl http://localhost:3000
curl http://localhost:8000/api
curl http://localhost:8080

# Testar o banco de dados internamente:
curl http://localhost:5432   # PostgreSQL
curl http://localhost:3306   # MySQL

# Ver regras de firewall:
sudo iptables -L
sudo ufw status
```

---

### FASE 5 — Problemas Comuns e Soluções

| Sintoma | Causa Provável | Solução |
|--------|---------------|---------|
| Página não carrega | Nginx/Apache parado | `sudo systemctl start nginx` |
| Página carrega mas sem dados | API parada ou com erro | Ver logs da API, reiniciar serviço |
| API funciona mas dados errados | Banco parado | `sudo systemctl start mysql` |
| Serviço reinicia e cai de novo | Erro de configuração | Verificar logs com `journalctl -xe` |
| "Connection refused" | Serviço não está ouvindo na porta | Verificar portas com `ss -tlnp` |
| "Permission denied" | Problema de permissão de arquivo | Verificar com `ls -la` |

---

### FASE 6 — Verificar Arquivos de Configuração

```bash
# Configuração do Nginx:
cat /etc/nginx/sites-enabled/*
cat /etc/nginx/nginx.conf

# Ver se a config do Nginx tem erros de sintaxe:
sudo nginx -t

# Configuração do banco (variáveis de ambiente da app):
cat /etc/environment
cat ~/.env
find /var/www/ -name ".env" 2>/dev/null
find /opt/ -name "*.conf" 2>/dev/null
```

---

### FASE 7 — Confirmar que Tudo Funciona

```bash
# Testar a aplicação pelo IP externo:
curl http://SEU-IP-AQUI

# Testar a API:
curl http://SEU-IP-AQUI/api

# Ver se todos os serviços estão ativos:
systemctl status nginx
systemctl status mysql   # ou postgresql
# (e o serviço da API)
```

---

## 📝 Como Documentar seu Diagnóstico

Ao final, você precisará explicar o que encontrou. Use este template:

---

### Template de Relatório

```
PROBLEMA IDENTIFICADO:
  Em qual camada: [Interface Web / API / Banco de Dados]
  Serviço afetado: [nginx / mysql / api-service / etc.]
  
SINTOMA OBSERVADO:
  [O que aparecia para o usuário - ex: "página em branco", "erro 502"]

CAUSA RAIZ:
  [Por que o serviço estava parado/falhando]
  [Exemplo: "O serviço MySQL estava inativo (status: failed)"]

AÇÃO CORRETIVA:
  [O que você fez para corrigir]
  [Exemplo: "Executei sudo systemctl start mysql"]

VALIDAÇÃO:
  [Como confirmou que estava funcionando]
  [Exemplo: "Acessei o endereço IP e os dados foram exibidos corretamente"]
```

---

## 🗺️ Sequência de Comandos Recomendada (Cola Rápida)

Execute **nesta ordem** ao entrar no servidor:

```bash
# 1. Ver serviços com falha
systemctl --failed

# 2. Status dos serviços principais
systemctl status nginx
systemctl status apache2
systemctl status mysql
systemctl status postgresql

# 3. Ver portas abertas
sudo ss -tlnp

# 4. Ver logs recentes do sistema
sudo journalctl -xe --no-pager | tail -50

# 5. Tentar acessar a aplicação localmente
curl -v http://localhost
curl -v http://localhost/api

# 6. Verificar configurações
sudo nginx -t
cat /etc/nginx/sites-enabled/*
```

---

## ❓ O Que Fazer Se Travar

1. **Copie a mensagem de erro** exatamente como aparece
2. **Identifique o serviço** que está causando o erro
3. **Verifique os logs** com `journalctl -u NOME-DO-SERVIÇO -n 50`
4. **Pesquise o erro** — erros Linux são muito documentados online

> [!TIP]
> O comando mais poderoso para iniciantes é `systemctl --failed`. Ele imediatamente mostra o que está quebrado, sem precisar adivinhar.

> [!IMPORTANT]
> Anote tudo que você fizer! O avaliador quer ver seu **raciocínio**, não só a solução. Explique por que você rodou cada comando.

---

*Guia criado para o Laboratório de Troubleshooting — Nível Iniciante*
