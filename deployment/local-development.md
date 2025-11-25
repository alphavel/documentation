# 🚀 Guia Rápido: Desenvolvimento Local com Alphavel

## Para Desenvolvedores Sem Swoole Instalado

### Início Rápido

```bash
# 1. Criar ou clonar projeto
composer create-project alphavel/skeleton meu-projeto
cd meu-projeto

# 2. Iniciar ambiente de desenvolvimento
make dev

# Ou manualmente:
docker-compose -f docker-compose.dev.yml up
```

**Primeira execução:** Pode levar 2-3 minutos (instalação automática do Swoole)  
**Próximas execuções:** Instantâneas

**Acesse:** http://localhost:9999

---

## O Que Acontece Automaticamente?

Quando você executa `make dev`, o container:

1. ✅ Instala dependências do sistema (curl, git, unzip, etc)
2. ✅ Instala extensões PHP necessárias (zip, etc)
3. ✅ Instala e ativa a extensão Swoole
4. ✅ Instala o Composer
5. ✅ Instala todas as dependências do projeto (`composer install`)
6. ✅ Cria estrutura de diretórios (storage, bootstrap/cache)
7. ✅ Define permissões corretas
8. ✅ Gera arquivo facades.php
9. ✅ Copia .env.example para .env
10. ✅ Inicia o servidor Swoole na porta 9999

---

## Comandos Úteis

### Via Makefile (Recomendado)

```bash
# Iniciar ambiente de desenvolvimento
make dev

# Parar ambiente
make dev-stop

# Ver logs em tempo real
make dev-logs

# Acessar shell do container
make dev-shell

# Reinstalar/reconstruir tudo
make dev-rebuild
```

### Via docker-compose

```bash
# Iniciar (em foreground, ver logs)
docker-compose -f docker-compose.dev.yml up

# Iniciar (em background)
docker-compose -f docker-compose.dev.yml up -d

# Parar
docker-compose -f docker-compose.dev.yml down

# Ver logs
docker-compose -f docker-compose.dev.yml logs -f app

# Acessar shell
docker-compose -f docker-compose.dev.yml exec app bash

# Executar comandos
docker-compose -f docker-compose.dev.yml exec app composer require alphavel/database
docker-compose -f docker-compose.dev.yml exec app php -v
```

---

## Estrutura de Portas

| Serviço | Porta Host | Porta Container | Descrição |
|---------|------------|-----------------|-----------|
| Aplicação | 9999 | 9999 | Servidor Swoole |
| MySQL | 3307 | 3306 | Banco de dados (dev) |

**Nota:** Porta 3307 no host para não conflitar com MySQL local

---

## Desenvolver no Container

### Instalar Pacotes

```bash
# Via make
make dev-shell
composer require alphavel/database

# Ou direto
docker-compose -f docker-compose.dev.yml exec app composer require alphavel/database
```

### Rodar Testes

```bash
docker-compose -f docker-compose.dev.yml exec app vendor/bin/phpunit
```

### Executar Scripts

```bash
docker-compose -f docker-compose.dev.yml exec app php artisan migrate
```

---

## Diferenças: Dev vs Production

### docker-compose.dev.yml (Desenvolvimento)
- ✅ Instala Swoole automaticamente
- ✅ Instala dependências automaticamente
- ✅ Usa imagem base `php:8.2-cli`
- ✅ Não requer build
- ✅ Volumes montados (código sincronizado)
- ✅ Porta 3307 para MySQL (evita conflito)
- ✅ Logs verbosos
- ⚠️ Primeira inicialização mais lenta

### docker-compose.yml (Produção)
- ✅ Build otimizado com Dockerfile
- ✅ Dependências já no build
- ✅ Imagem pronta para produção
- ✅ Mais rápido em execução
- ✅ Porta 3306 padrão para MySQL
- ⚠️ Requer rebuild após mudanças no código

---

## Solução de Problemas

### Container não inicia / trava na instalação

```bash
# Ver logs detalhados
docker-compose -f docker-compose.dev.yml logs -f

# Reconstruir do zero
docker-compose -f docker-compose.dev.yml down -v
docker-compose -f docker-compose.dev.yml up
```

### Erro de permissões

```bash
# Dentro do container
docker-compose -f docker-compose.dev.yml exec app bash
chmod -R 777 storage bootstrap/cache
```

### Swoole não foi instalado

```bash
# Forçar reinstalação
docker-compose -f docker-compose.dev.yml exec app pecl install swoole
docker-compose -f docker-compose.dev.yml exec app docker-php-ext-enable swoole
docker-compose -f docker-compose.dev.yml restart app
```

### Porta 8080 já está em uso

Edite `.env` e mude a porta:

```env
APP_PORT=8081
```

Ou especifique ao iniciar:

```bash
APP_PORT=8081 docker-compose -f docker-compose.dev.yml up
```

---

## Limpar Tudo

```bash
# Parar e remover containers + volumes
docker-compose -f docker-compose.dev.yml down -v

# Remover vendor e cache locais
rm -rf vendor storage/cache/* storage/logs/* bootstrap/cache/*
```

---

## Comparação de Workflows

### Sem docker-compose.dev.yml (Antigo)

```bash
# 1. Instalar Swoole na máquina (complexo)
sudo pecl install swoole

# 2. Configurar PHP
echo "extension=swoole.so" >> /etc/php/8.2/cli/conf.d/20-swoole.ini

# 3. Instalar dependências
composer install

# 4. Configurar ambiente
cp .env.example .env
mkdir -p storage/framework storage/logs storage/cache bootstrap/cache
chmod -R 777 storage bootstrap/cache

# 5. Iniciar servidor
php public/index.php
```

**Problemas:**
- ❌ Swoole pode não funcionar no macOS/Windows
- ❌ Conflitos com outras versões do PHP
- ❌ Configuração varia por SO
- ❌ ~10-15 minutos de setup manual

### Com docker-compose.dev.yml (Novo)

```bash
# 1. Iniciar (tudo automático)
make dev
```

**Benefícios:**
- ✅ Funciona em qualquer SO (Linux, macOS, Windows)
- ✅ Ambiente isolado e consistente
- ✅ Sem conflitos com instalações locais
- ✅ ~2-3 minutos de setup automático
- ✅ Fácil de compartilhar com time

---

## Boas Práticas

### Para Desenvolvimento Diário

1. Use sempre `make dev` ou `docker-compose -f docker-compose.dev.yml up`
2. Mantenha o container rodando (não recrie a cada mudança)
3. Código é sincronizado automaticamente via volumes
4. Para mudanças no composer.json, execute `composer install` dentro do container

### Para Commitar Código

1. Não comite arquivos gerados (vendor, storage/*, etc)
2. .gitignore já está configurado corretamente
3. Outros desenvolvedores usarão o mesmo docker-compose.dev.yml

### Para CI/CD

1. Use `docker-compose.yml` (produção) no pipeline
2. `docker-compose.dev.yml` é apenas para desenvolvimento local
3. Testes podem rodar em qualquer dos dois ambientes

---

## FAQ

**Q: Preciso instalar Swoole na minha máquina?**  
A: Não! O docker-compose.dev.yml instala tudo dentro do container.

**Q: As mudanças no código são refletidas automaticamente?**  
A: Sim! O código está montado via volume, mudanças são instantâneas.

**Q: Posso usar meu IDE favorito?**  
A: Sim! Edite os arquivos normalmente. O container apenas executa o código.

**Q: Como depurar o código?**  
A: Configure Xdebug (instruções separadas) ou use `var_dump()` e veja nos logs.

**Q: O banco de dados é persistente?**  
A: Sim! Os dados ficam em volume Docker (`dbdata-dev`).

**Q: Posso usar Redis/Postgres?**  
A: Sim! Adicione mais serviços no docker-compose.dev.yml conforme necessário.

**Q: É mais lento que execução local?**  
A: Não significativamente. Swoole compensa com performance superior.

**Q: Funciona no Windows?**  
A: Sim! Desde que tenha Docker Desktop instalado.

---

## Próximos Passos

1. ✅ Inicie o ambiente: `make dev`
2. ✅ Acesse http://localhost:9999
3. ✅ Leia a documentação completa no README.md
4. ✅ Instale pacotes adicionais: `composer require alphavel/database`
5. ✅ Comece a desenvolver!

---

**Dúvidas?** Veja a documentação completa no [README.md](README.md)  
**Problemas?** Abra uma issue no [GitHub](https://github.com/alphavel/skeleton/issues)
