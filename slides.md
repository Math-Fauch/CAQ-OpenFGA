---
theme: default
title: OpenFGA — Authorization as a Graph
info: |
  A small Slidev proof of concept for a technical OpenFGA talk.
transition: slide-left
mdc: true
addons:
  - slidev-addon-excalidraw
colorSchema: dark
drawings:
  persist: false
---

<div class="eyebrow">Mathieu Faucher</div>

# Autorisation 101

<p class="lede">Modélisez des relations. Posez une seule question.<br>Gardez les politiques hors du code applicatif.</p>

---
layout: two-cols-header
---

<div class="eyebrow">00 · WHOAMI</div>

# Mathieu Faucher

::left::

- Développeur chez Botpress
- Je travaille actuellement sur l’autorisation
- Migration de 37 000+ workspaces vers OpenFGA

::right::

<img src="/BP_Brandmark_Black.png" alt="Botpress" style="width: 12rem" />

---
layout: two-cols-header
layoutClass: concrete-example
---

<div class="eyebrow">01 · LE PROBLEME</div>

# Exemple concret

::left::


## <b>Question</b>

<div class="big-quote">
“Can <strong>Alice</strong> view the roadmap?”
</div>

<div class="question-parts">
  <div><span>user</span><b>alice</b></div>
  <div><span>relation</span><b>viewer</b></div>
  <div><span>object</span><b>document:roadmap</b></div>
</div>

::right::

## <div v-click><b>Objets</b></div>

<div class="rule-stack">
  <div v-click class="rule-card">User</div>
  <div v-click class="rule-card">Team</div>
  <div v-click class="rule-card">Folder</div>
  <div v-click class="rule-card">Document</div>
</div>

---
layout: center
layoutClass: paths-slide
---

<div class="eyebrow">02 · Les chemins</div>

<div class="paths-diagram">
  <svg viewBox="0 0 640 300" role="img" aria-label="Trois chemins entre Alice et le document roadmap">
    <g v-click class="graph-path graph-path--direct">
      <path d="M 130 150 H 260 M 380 150 H 510" />
      <rect x="260" y="128" width="120" height="44" rx="10" />
      <text x="320" y="155">Accès direct</text>
    </g>
    <g v-click class="graph-path graph-path--folder">
      <path d="M 130 143 H 190 V 68 H 270 M 370 68 H 450 V 143 H 510" />
      <rect x="270" y="45" width="100" height="46" rx="10" />
      <text x="320" y="73">Folder</text>
      <text class="path-label" x="208" y="57">Accès par un dossier</text>
    </g>
    <g v-click class="graph-path graph-path--team">
      <path d="M 130 157 H 190 V 232 H 270 M 370 232 H 450 V 157 H 510" />
      <rect x="270" y="209" width="100" height="46" rx="10" />
      <text x="320" y="237">Team</text>
      <text class="path-label" x="202" y="250">Accès par une équipe</text>
    </g>
    <g class="graph-endpoint">
      <rect x="30" y="125" width="100" height="50" rx="12" />
      <text x="80" y="155">Alice</text>
    </g>
    <g class="graph-endpoint">
      <rect x="510" y="125" width="100" height="50" rx="12" />
      <text x="560" y="147">document:</text>
      <text x="560" y="162">roadmap</text>
    </g>
  </svg>

  <div v-click class="graph-takeaway"><span>→</span> Un seul graphe pour toutes ces questions</div>
</div>

---
layout: two-cols-header
---

<div class="eyebrow">03 · Le model</div>

::left::

```yaml {all|1-2|4,6,10,17|12-15|all}
model
  schema 1.1

type user

type team
  relations
    define member: [user]

type folder
  relations
    define owner: [user]
    define parent: [folder]
    define editor: [user, team#member] or owner or editor from parent
    define viewer: [user, team#member] or editor or viewer from parent

type document
  relations
    define owner: [user]
    define folder: [folder]
    define editor: [user, team#member] or owner or editor from parent
    define viewer: [user, team#member] or editor or viewer from parent
```

::right::

<div class="model-notes">
  <div v-click>
    <b>Types</b>
    <span>Le nom dans le domaine</span>
  </div>
  <div v-click>
    <b>Relations</b>
    <span>comment lier ces noms</span>
  </div>
  <div v-click>
    <b>définition</b>
    <span>instance de liaison</span>
  </div>
</div>

<div v-click class="callout">La politique devient versionné et testable.</div>

---

<div class="eyebrow">04 · Modèles</div>

- ACL (Access Control List)
- RBAC (Role-Based Access Control)
- ABAC (Attribute-Based Access Control)

---
layout: two-cols-header
layoutClass: comparison-slide
---

<div class="eyebrow">05 · ACL</div>

::left::

<div class="comparison-label">TypeScript</div>

```ts
function can_read(user: string, document: string) {
  return acl[document]?.read?.includes(user) ?? false  
}
```

::right::

<div class="comparison-label">OpenFGA</div>

```yaml {all}
model
  schema 1.1

type user

type team
  relations
    define member: [user]

type folder
  relations
    define owner: [user]
    define parent: [folder]
    define editor: [user, team#member] or owner or editor from parent
    define viewer: [user, team#member] or editor or viewer from parent

type document
  relations
    define owner: [user]
    define folder: [folder]
    define editor: [user, team#member] or owner or editor from parent
    define viewer: [user, team#member] or editor or viewer from parent
```



---
layout: two-cols-header
layoutClass: comparison-slide
---

<div class="eyebrow">06 · RBAC</div>

::left::

<div class="comparison-label">TypeScript</div>

```ts {all|1|3-7|6|8-18|all}
type Role = "admin" | "editor" | "viewer"

type User = {
  id: string
  name: string
  role: Role
}

function canDeleteArticles(user: User) {
  return user.role === "admin"
}

function canEditArticles(user: User) {
  return user.role === "admin" || user.role === "editor"
}

function canViewArticles(user: User) {
  return true // Everyone can view
}
```

::right::

<div class="comparison-label">OpenFGA</div>

```yaml {all|6-10|16,17,23,24|all}
model
  schema 1.1

type user

type team
  relations
    define admin: [user]
    define editor: [user] or admin
    define viewer: [user] or editor

type folder
  relations
    define owner: [user]
    define parent: [folder]
    define editor: [user, team#editor, team#admin] or owner or editor from parent
    define viewer: [user, team#viewer, team#admin] or editor or viewer from parent

type document
  relations
    define owner: [user]
    define folder: [folder]
    define editor: [user, team#editor, team#admin] or owner or editor from parent
    define viewer: [user, team#viewer, team#admin] or editor or viewer from parent
```

---
layout: two-cols-header
layoutClass: comparison-slide
---

<div class="eyebrow">07 · ABAC</div>

# Make the Playground earn its screen time.

::left::

<div class="comparison-label">TypeScript</div>

``` ts {all|1-6|7-10|11-23|all}
type Grant = {
  user: string
  document: string
  relation: "editor" | "viewer"
  expiresAt: number
}
const GRANTS: Grant[] = [
  { user: "alice", document: "roadmap", relation: "viewer", expiresAt: 1700000000 },
  { user: "bob", document: "roadmap", relation: "editor", expiresAt: 1700000000 },
]
function can(user: string, relation: "editor" | "viewer", document: string) {
  for (const grant of GRANTS) {
    if (
      grant.user === user &&
      grant.relation === relation &&
      grant.document === document &&
      Date.now() < grant.expiresAt
    ) {
      return true
    }
  }
  return false
}
```

::right::

<div class="comparison-label">OpenFGA</div>

```yaml {all|16-18|10-14|all}
model
  schema 1.1

type user

type team
  relations
    define member: [user]

type document
  relations
    define owner: [user]
    define editor: [user with non_expired_grant]
    define viewer: [user with non_expired_grant]

condition non_expired_grant(current_time: timestamp, grant_time: timestamp, grant_duration: duration) {
  current_time < grant_time + grant_duration
}
```

---

<div class="eyebrow">08 · Trucs</div>

- Toujours garder une seule source de vérité
- Ajouter un type role pour faire des role personnalisés
- Utiliser le flag `user:*`

---
layout: center
class: takeaway-slide
---

<div class="eyebrow">EN RÉSUMÉ</div>

# Modélisez les relations.<br><span>Vérifiez les autorisations.</span>

<div class="takeaways">
  <div><b>01</b><span>Politique centralisée</span></div>
  <div><b>02</b><span>Relations composables</span></div>
  <div><b>03</b><span>Décisions explicables</span></div>
</div>

<a href="https://openfga.dev" target="_blank" class="final-link">openfga.dev ↗</a>

<div class="final-question">Des questions ?</div>
