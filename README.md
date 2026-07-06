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

## 2. Ajustar as regras de segurança do Firestore

Por padrão, o "modo de teste" expira em 30 dias. Para o site funcionar o mês inteiro (e depois, se quiserem reutilizar), vá em **Firestore Database > Regras** e substitua pelo conteúdo abaixo, depois clique em **Publicar**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /escala/julho2026 {
      allow read, write: if true;
    }
  }
}
```

Isso libera leitura e escrita apenas para o documento específico da escala (não para o banco inteiro). Como é uma ferramenta interna sem dados sensíveis, essa regra simples é suficiente — mas qualquer pessoa com o link do site poderá editar a escala, então compartilhem o link só com o grupo.

## 3. Publicar no Vercel (gratuito)

**Opção mais simples — pelo site, sem instalar nada:**

1. Crie uma conta gratuita em https://vercel.com (pode entrar com GitHub, GitLab ou e-mail).
2. Suba esta pasta (`escala-usg-vercel`) para um repositório no GitHub:
   - Crie um repositório novo em https://github.com/new
   - Envie os 3 arquivos (`index.html`, `firebase-config.js`, `README.md`) para esse repositório (pelo site do GitHub mesmo, em **Add file > Upload files**, é o caminho mais rápido).
3. No Vercel, clique em **Add New > Project**, selecione o repositório que você acabou de criar e clique em **Deploy**.
   - Não precisa mudar nenhuma configuração — o Vercel detecta que é um site estático automaticamente.
4. Em ~30 segundos o Vercel te dá uma URL pública (tipo `escala-usg.vercel.app`). Esse é o link para compartilhar com o Pedro, Narelly, Thiago, Júlio e Luís.

**Alternativa via terminal (se algum de vocês tiver Node.js instalado):**

```bash
npm i -g vercel
cd escala-usg-vercel
vercel --prod
```

Siga as instruções na tela (login e confirmação do diretório) e o próprio comando já devolve a URL pública.

## 4. Testar

Abra a URL publicada em duas abas diferentes (ou peça para outro residente abrir) e adicione uma agenda em uma aba — ela deve aparecer automaticamente na outra em poucos segundos, confirmando que o Firestore está sincronizando.

## Se algo der errado

- **"Firebase não configurado"** aparece no site: os valores em `firebase-config.js` ainda estão como `"COLE_AQUI"`.
- **"Erro ao conectar ao Firebase"**: confira se o Firestore foi criado (passo 1.3) e se as regras foram publicadas (passo 2).
- Qualquer mudança feita depois no `firebase-config.js` ou `index.html` exige um novo commit no GitHub — o Vercel republica sozinho a cada push.
