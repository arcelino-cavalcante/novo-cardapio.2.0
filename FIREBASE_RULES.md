# Firestore Security Rules

Use o console do Firebase para abrir **Firestore Database → Regras** e substitua o conteúdo pelas regras abaixo. Leituras permanecem públicas, porém as escrituras só são liberadas para usuários autenticados via Firebase Authentication (Email/Senha) cujo UID esteja listado no array `allowedEditorUids`. Ajuste a lista antes de publicar. Recomendo também ativar o **Firebase App Check** para o app web, reforçando que apenas seu front-end aprovado consiga fazer chamadas.

```rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Documento do cardápio público
    match /cardapio/conteudo {
      allow read: if true; // menu visível para todos

      allow write: if isEditor(request.auth) && isValidPayload(request.resource.data);

      function allowedEditorUids() {
        // Substitua pelos UIDs dos emails cadastrados no Firebase Auth que podem acessar o painel
        return [
          'UID_DO_USUARIO_ADMIN_1',
          'UID_DO_USUARIO_ADMIN_2'
        ];
      }

      function isEditor(auth) {
        return auth != null && allowedEditorUids().hasAny([auth.uid]);
      }

      function isValidPayload(data) {
        return data.keys().hasOnly(['categories', 'products', 'fees', 'monteSeu', 'optionGroups', 'info', 'sizeLabels'])
          && data.categories is list
          && data.products is list
          && data.monteSeu is list
          && data.optionGroups is list
          && data.info is map
          && data.info.keys().hasOnly(['description', 'address', 'whatsapp', 'instagram', 'open'])
          && data.info.open is bool
          && data.sizeLabels is map
          && data.sizeLabels.keys().hasOnly(['p', 'm', 'g'])
          && data.fees is map
          && data.fees.keys().hasOnly(['base', 'sitios'])
          && data.fees.base is number
          && data.fees.sitios is list
          && data.categories.size() <= 50
          && data.products.size() <= 500
          && data.monteSeu.size() <= 50
          && data.optionGroups.size() <= 50
          && data.fees.sitios.size() <= 100;
      }
    }

    // Bloqueia todo o restante por padrão.
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

> ✅ Atualize `allowedEditorUids()` com os UIDs reais (console Firebase → Authentication → usuários → copiar UID). No front-end do painel, garanta que o usuário faça login via Firebase Auth antes de acessar `admin.html`; o SDK do Firestore enviará automaticamente o `request.auth`. Combine com App Check para proteção adicional.
Site para urls: https://postimages.org/