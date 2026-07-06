# Escala USG

Site estático (HTML puro) com dados sincronizados em tempo real via Firebase Firestore.
Hospedagem gratuita no Vercel.

## Funcionalidades

- Escala semanal por turno (manhã/tarde), com agendas de USG cadastradas em cada turno.
- Atribuição de residente por agenda, com contador de turnos cobertos e relatório mensal.
- Renomear uma agenda depois de criada (ícone de lápis).
- Copiar os títulos das agendas de uma semana para a semana seguinte (residentes não são copiados — o turno entra livre).
- Criar a escala de um novo mês a qualquer momento (botão "+ novo mês"); as semanas são geradas automaticamente.
- Adicionar ou remover residentes da equipe (botão "+ residente" e "×" em cada nome).
- **Login com Google exigido apenas para editar** — a visualização de toda a escala e dos relatórios é livre para qualquer pessoa, sem login.
- **Instalável como app (PWA)** — dá para adicionar à tela inicial do celular e abrir como um aplicativo, sem barra de navegador.

## Instalar como app no celular

Depois de publicado no Vercel, cada residente pode instalar o site como se fosse um app:

- **Android (Chrome)**: abra o link, toque no menu (⋮) e escolha **"Instalar app"** ou **"Adicionar à tela inicial"**.
- **iPhone (Safari)**: abra o link, toque no ícone de compartilhar (□ com seta para cima) e escolha **"Adicionar à Tela de Início"**.

O ícone aparece na tela inicial e o site abre em tela cheia, sem a barra de endereço do navegador.

## 1. Criar o projeto no Firebase (gratuito)

1. Acesse https://console.firebase.google.com e clique em **Adicionar projeto**.
2. Dê um nome (ex.: `escala-usg`) e conclua a criação (pode desativar o Google Analytics, não é necessário).
3. No menu lateral, clique em **Compilação > Firestore Database** > **Criar banco de dados**.
   - Escolha a localização mais próxima (ex.: `southamerica-east1` para o Brasil).
   - Selecione **Iniciar em modo de teste** (ajustaremos as regras no passo 3).
4. No menu lateral, clique no ícone de engrenagem > **Configurações do projeto**.
5. Em **Seus aplicativos**, clique no ícone **</>** (Web) para registrar um app.
   - Dê um apelido (ex.: `escala-web`) e clique em **Registrar app**. Não precisa configurar hosting do Firebase.
6. Copie o objeto `firebaseConfig` que aparece na tela e cole no arquivo `firebase-config.js` deste projeto, substituindo os valores `"COLE_AQUI"`.

## 2. Ativar o login com Google

A visualização do site é livre para todos, sem login. Mas para editar (adicionar agenda, atribuir residente, etc.) é preciso entrar com uma conta Google.

1. No menu lateral do Firebase, clique em **Compilação > Authentication** > **Vamos começar** (ou **Get started**).
2. Na aba **Sign-in method** (Método de login), clique em **Google** na lista de provedores.
3. Ative o toggle **Enable/Ativar**, escolha um e-mail de suporte do projeto e clique em **Salvar**.
4. Ainda em Authentication, vá em **Settings > Authorized domains** (Domínios autorizados) e clique em **Add domain**. Adicione o domínio que o Vercel vai te dar (ex.: `escala-usg.vercel.app`) — sem isso, o login com Google não funciona fora do `localhost`. Você faz esse passo depois do passo 3 (publicar no Vercel), quando já souber a URL final.

## 3. Ajustar as regras de segurança do Firestore

Por padrão, o "modo de teste" expira em 30 dias. Vá em **Firestore Database > Regras** e substitua pelo conteúdo abaixo, depois clique em **Publicar**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /escala/julho2026 {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

Isso libera a **leitura para qualquer pessoa** (sem login) e exige **login com Google para escrever** (editar a escala). Qualquer pessoa com uma conta Google pode editar — não há lista de e-mails permitidos; se quiserem restringir a apenas os 5 e-mails de vocês, me avisem que eu ajusto a regra.

## 4. Publicar no Vercel (gratuito)

**Opção mais simples — pelo site, sem instalar nada:**

1. Crie uma conta gratuita em https://vercel.com (pode entrar com GitHub, GitLab ou e-mail).
2. Suba esta pasta (`escala-usg-vercel`) para um repositório no GitHub:
   - Crie um repositório novo em https://github.com/new
   - Envie os arquivos (`index.html`, `firebase-config.js`, `README.md`) para esse repositório (pelo site do GitHub mesmo, em **Add file > Upload files**, é o caminho mais rápido).
3. No Vercel, clique em **Add New > Project**, selecione o repositório que você acabou de criar e clique em **Deploy**.
   - Não precisa mudar nenhuma configuração — o Vercel detecta que é um site estático automaticamente.
4. Em ~30 segundos o Vercel te dá uma URL pública (tipo `escala-usg.vercel.app`). Esse é o link para compartilhar com o Pedro, Narelly, Thiago, Júlio e Luís.
5. Volte no Firebase (passo 2.4) e adicione essa URL como domínio autorizado do Authentication — sem isso o botão de login com Google não funciona no site publicado.

**Alternativa via terminal (se algum de vocês tiver Node.js instalado):**

```bash
npm i -g vercel
cd escala-usg-vercel
vercel --prod
```

Siga as instruções na tela (login e confirmação do diretório) e o próprio comando já devolve a URL pública.

## 5. Testar

Abra a URL publicada em duas abas diferentes (ou peça para outro residente abrir) e adicione uma agenda em uma aba — ela deve aparecer automaticamente na outra em poucos segundos, confirmando que o Firestore está sincronizando. Teste também o botão "Entrar com Google" — sem estar logado, qualquer tentativa de editar deve mostrar um aviso pedindo login; visualizar a escala e os relatórios continua funcionando normalmente sem login.

## Se algo der errado

- **"Firebase não configurado"** aparece no site: os valores em `firebase-config.js` ainda estão como `"COLE_AQUI"`.
- **"Erro ao conectar ao Firebase"**: confira se o Firestore foi criado (passo 1.3) e se as regras foram publicadas (passo 3).
- **Botão de login com Google não abre nada / dá erro de domínio**: falta adicionar a URL do Vercel em Authentication > Settings > Authorized domains (passo 2.4 / 4.5).
- **Login funciona mas a edição continua bloqueada**: confira se as regras do Firestore (passo 3) foram publicadas com `allow write: if request.auth != null;`.
- Qualquer mudança feita depois no `firebase-config.js` ou `index.html` exige um novo commit no GitHub — o Vercel republica sozinho a cada push.
