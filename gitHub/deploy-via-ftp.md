# Deploy Automático via FTP com GitHub Actions

Este documento explica como configurar um deploy automático via FTP usando GitHub Actions. Inclui a criação de chaves secretas e recomendações de segurança.
---

## 📦 O que é isso?

Um workflow GitHub Actions pode ser configurado para enviar seus arquivos para um servidor FTP automaticamente toda vez que você fizer `push` para o branch principal (`main`, por exemplo).
---
## ⚙️ Como funciona?

1. O GitHub Actions roda um workflow sempre que há um `push` no branch configurado
2. Esse workflow faz checkout do código
3. Os arquivos são enviados para o servidor FTP usando a ação `SamKirkland/FTP-Deploy-Action`
---
## 📁 Exemplo de arquivo de workflow

Crie o arquivo `.github/workflows/deploy.yml` com o seguinte conteúdo:

```yaml
name: Deploy via FTP

on:
  push:
    branches:
      - main  # Altere para o branch que deseja monitorar

jobs:
  ftp-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout do código
        uses: actions/checkout@v3
        with:
          fetch-depth: 2  # Recomendado para obter todos os arquivos alterados

      - name: Enviar via FTP
        uses: SamKirkland/FTP-Deploy-Action@v4.3.4
        with:
          server: ${{ secrets.FTP_HOST }}
          username: ${{ secrets.FTP_USERNAME }}
          password: ${{ secrets.FTP_PASSWORD }}
          local-dir: ./  # Diretório local a ser enviado (altere conforme necessário)
          server-dir: /caminho/no/servidor/  # Diretório no servidor (altere conforme necessário)
          exclude: |
            **/.git*
            **/.github*
            **/node_modules/**
            .env
            README.md
```
---

## 🔐 Como criar e usar Secrets

Secrets são variáveis protegidas armazenadas no repositório do GitHub e utilizadas no workflow para proteger informações sensíveis.

### Passos para criar secrets:

1. Acesse o seu repositório no GitHub
2. Vá em **Settings > Secrets and variables > Actions**
3. Clique em **New repository secret**
4. Crie os seguintes secrets:
   * `FTP_HOST`: o endereço do servidor FTP (ex: `ftp.seusite.com`)
   * `FTP_USERNAME`: o nome de usuário do FTP
   * `FTP_PASSWORD`: a senha do usuário FTP

> **IMPORTANTE**: Para cada novo projeto, você precisará adicionar novamente estas secrets, pois elas são específicas para cada repositório.
---

## ⚠️ Configuração por projeto

Para cada novo projeto que você criar, é necessário:

1. Criar o arquivo `.github/workflows/deploy.yml` no repositório
2. Adicionar as secrets mencionadas acima nas configurações do repositório
3. Modificar as seguintes configurações no arquivo de workflow de acordo com o projeto:
   * `local-dir`: O diretório local que contém os arquivos a serem enviados
     * Exemplos: `./` (todo o repositório), `./dist/` (apenas o diretório dist)
   * `server-dir`: O caminho no servidor FTP onde os arquivos serão enviados
     * Exemplo: `/public_html/meusite/` ou `/www/dominio.com/`
   * `branches`: Configurar qual branch ativa o deploy (por padrão é `main`)
---

## 🔐 Segurança das Secrets

* As secrets são **criptografadas** e **não ficam visíveis** nem mesmo para administradores do repositório
* Nunca insira senhas diretamente no arquivo `.yml` — use sempre `secrets`
* Evite expor arquivos sensíveis como `.env`, `.git`, etc., usando as regras de exclusão no workflow
---
## 📄 Arquivos a ignorar

O exemplo acima já inclui exclusões básicas no próprio workflow. Alternativamente, você pode criar um arquivo `.ftp-deploy-ignore` na raiz do projeto:

```
**/.git*
**/.github*
**/node_modules/**
.env
README.md
package-lock.json
```
---
## ✅ Dicas finais

* Teste seu FTP manualmente antes de automatizar com GitHub Actions
* Certifique-se de que os caminhos (`local-dir` e `server-dir`) estão corretos
* Use um branch exclusivo para deploy, se preferir separar da branch principal
* Para sites que executam PHP ou outros scripts de servidor, verifique as permissões de arquivo após o upload
* Considere adicionar uma etapa de build antes do deploy para projetos que precisam ser compiladosm

---
