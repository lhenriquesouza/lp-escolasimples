# Plano: Wire LP → Core Checkout (Formulario de Coleta)

**Data**: 2026-06-11
**Repo**: lp-escolasimples
**Depende de**: core-escolasimples rodando (checkout.js pronto)

---

## Escopo

Adicionar formulario de coleta na landing page. Ao clicar "Comecar agora" em
qualquer card de plano, abre um modal com os campos do cliente. Submit →
redirect GET para o core checkout.

---

## Design Decisions

| Decisao | Escolha | Motivo |
|---|---|---|
| Formato | Modal (dialog nativo) | Mantem contexto da LP, nao sai da pagina |
| Metodo | GET redirect (query string) | Sem CORS, sem JS fetch, sem build step |
| URL producao vs local | Auto-deteccao via `window.location` | Mesmo HTML funciona nos 2 ambientes |
| Plano | `data-plan` no botao | Plano injetado automaticamente, zero logica condicional |
| Validacao | `required` + `pattern` no HTML | Navegador valida nativo, sem JS extra |
| Estilo | Tokens oklch existentes | Mesmo design system da LP |

---

## Campos

| Campo | name | Tipo | Obrigatorio | Notas |
|---|---|---|---|---|
| Nome completo | `name` | text | sim | |
| Email | `email` | email | sim | |
| Nome da escola | `school` | text | sim | vira slug no core |
| Cidade | `city` | text | sim | |
| Estado | `state` | select (27 UFs) | sim | |
| Celular/WhatsApp | `phone` | tel | nao | |
| CNPJ | `cnpj` | text | nao | pattern: `\d{14}` |

---

## Passo a passo

### 1. Adicionar CHECKOUT_BASE no `<head>`

```html
<script>
  (() => {
    const proto = window.location.protocol;
    const host = window.location.hostname;
    window.CHECKOUT_BASE =
      proto === 'file:' || host === 'localhost' || host === '127.0.0.1'
        ? 'http://localhost:3001'
        : 'https://api.escolasimples.com';
  })();
</script>
```

### 2. Adicionar modal HTML

Dialog nativo `<dialog>` com formulario, antes do `</body>`:

```html
<dialog id="checkout-modal">
  <div class="modal-inner">
    <button class="modal-close" onclick="this.closest('dialog').close()">&times;</button>
    <h2>Comece agora</h2>
    <p class="modal-plan">Plano <strong id="modal-plan-name">Essencial</strong></p>

    <form id="checkout-form" method="GET">
      <input type="hidden" name="plan" id="modal-plan-input">

      <label>
        Nome completo *
        <input type="text" name="name" required autocomplete="name">
      </label>

      <label>
        Email *
        <input type="email" name="email" required autocomplete="email">
      </label>

      <label>
        Nome da escola *
        <input type="text" name="school" required>
      </label>

      <div class="modal-row">
        <label>
          Cidade *
          <input type="text" name="city" required autocomplete="address-level2">
        </label>
        <label>
          Estado *
          <select name="state" required>
            <option value="">UF</option>
            <option>AC</option><!-- ... 27 UFs ... -->
          </select>
        </label>
      </div>

      <label>
        Celular / WhatsApp
        <input type="tel" name="phone" autocomplete="tel">
      </label>

      <label>
        CNPJ (opcional)
        <input type="text" name="cnpj" pattern="\d{14}" placeholder="Apenas números">
      </label>

      <button type="submit" class="btn btn-primary btn-full">Continuar para checkout</button>
    </form>
  </div>
</dialog>
```

### 3. Adicionar JS para abrir modal

```html
<script>
  // Abrir modal ao clicar "Começar agora"
  document.querySelectorAll('.price-card .btn, [data-plan]').forEach(btn => {
    btn.addEventListener('click', (e) => {
      e.preventDefault();
      const plan = btn.dataset.plan || 
                   btn.closest('.price-card')?.querySelector('h3')?.textContent
                     .toLowerCase().normalize('NFD').replace(/[^a-z]/g, '');
      document.getElementById('modal-plan-name').textContent =
        plan.charAt(0).toUpperCase() + plan.slice(1);
      document.getElementById('modal-plan-input').value = plan;
      document.getElementById('checkout-modal').showModal();
    });
  });

  // Submit → redirect para core
  document.getElementById('checkout-form').addEventListener('submit', (e) => {
    e.preventDefault();
    const form = e.target;
    const params = new URLSearchParams(new FormData(form));
    window.location.href = CHECKOUT_BASE + '/checkout?' + params.toString();
  });

  // Fechar ao clicar fora (backdrop)
  document.getElementById('checkout-modal').addEventListener('click', (e) => {
    if (e.target === e.currentTarget) e.currentTarget.close();
  });
</script>
```

### 4. Adicionar `data-plan` nos botoes

```html
<!-- Essencial -->
<a href="#" class="btn btn-ghost" data-plan="essencial">Começar agora</a>

<!-- Profissional -->
<a href="#" class="btn btn-primary" data-plan="profissional">Começar agora</a>

<!-- Avançado -->
<a href="#" class="btn btn-ghost" data-plan="avancado">Começar agora</a>
```

### 5. CSS do modal

Usar tokens existentes (`--surface`, `--border`, `--radius-2`, etc.):

```css
#checkout-modal {
  border: none;
  border-radius: var(--radius-2);
  padding: 0;
  max-width: 480px;
  width: 90%;
  background: var(--surface);
  box-shadow: var(--shadow-2);
}
#checkout-modal::backdrop {
  background: rgba(0,0,0,0.5);
}
.modal-inner { padding: 2rem; }
.modal-close { /* botao X no canto */ }
/* labels, inputs, layout do form */
```

---

## Arquivos tocados

| Arquivo | Acao |
|---|---|
| `lp-escolasimples/index.html` | Unico arquivo — adicionar modal HTML + CSS + JS inline |

---

## Como testar localmente

```bash
# 1. Garantir que o core esta rodando
cd ~/core-escolasimples
docker ps | grep core || docker run -d --name core --network host ...

# 2. Abrir LP no navegador
xdg-open ~/lp-escolasimples/index.html

# 3. Clicar "Começar agora" → preencher form → submit
#    Deve redirecionar para http://localhost:3001/checkout?plan=essencial&name=...

# 4. Tela de confirmacao do core aparece
#    Confirmar → redirect pra /checkout/success
```

---

## Validacao pre-deploy

- [ ] Modal abre ao clicar nos 3 planos (essencial, profissional, avancado)
- [ ] Campos obrigatorios bloqueiam submit vazio
- [ ] CNPJ invalido (nao-numerico) bloqueia submit
- [ ] Submit redireciona com query string correta
- [ ] Tela de confirmacao do core mostra dados recebidos
- [ ] Abrir via `file://` → aponta `localhost:3001`
- [ ] Deploy GitHub Pages → aponta `api.escolasimples.com`

---

## Nao fazer agora

- Integracao real com AbacatePay (precisa de API key + conta aprovada)
- Email de boas-vindas pos-provisioning
- Validacao de subdominio em tempo real (fetch → `/api/check-subdomain`)
