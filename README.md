# MarketFaesa

[MarketFaesa](https://arthurnunesdev.github.io/Projeto-Integrador-Computacional/) web desenvolvido no Projeto Integrador Computacional da FAESA. O frontend é uma SPA em React publicada no GitHub Pages; o backend em Java com Spring Boot está começando agora e ainda não se comunica com o front.

## Funcionalidades

- **Login e cadastro**: telas simples, com troca direta entre login e cadastro. Por enquanto a autenticação é simulada no navegador e só funciona em `npm run dev`: copie `MarketPlace/.env.example` para `MarketPlace/.env.development.local` e defina um usuário/senha de teste. No build de produção o login fica desativado até a API existir. O formulário de cadastro valida os campos, mas ainda não salva nada. O botão "Sair", no menu do avatar do cabeçalho, encerra a sessão.
- **Perfil**: página com os dados do usuário logado.
- **Configurações**: tema claro/escuro, privacidade (perfil público, mostrar e-mail, permitir mensagens), notificações e a opção de reduzir animações.
- **Persistência local**: sessão e preferências ficam no `localStorage` (`marketfaesa-auth`, `marketfaesa-theme`, `marketfaesa-config`). Ainda não há chamadas HTTP.

## Tecnologias

| Camada | Stack |
|---|---|
| Frontend | React 19, Vite 8, Tailwind CSS 4, ESLint (JavaScript/JSX) |
| Backend | Java 21+, Spring Boot 4.1 (Spring Web MVC, Spring Data JPA), Maven (via Maven Wrapper) |
| Banco | PostgreSQL 16 no [Neon](https://neon.tech), migrations com Flyway |
| Deploy | GitHub Actions + GitHub Pages |

## Arquitetura

```
MarketFaesaWeb/
├── .github/
│   ├── workflows/deploy.yml         # build e deploy do front no GitHub Pages
│   └── pull_request_template.md     # modelo de descrição de PR
├── MarketPlace/                     # frontend (React + Vite)
│   ├── public/Imagens/              # ícones SVG, referenciados por caminho (ex.: ./Imagens/Sino.svg)
│   ├── src/
│   │   ├── main.jsx                 # ponto de entrada
│   │   ├── App.jsx                  # estado global: tema, autenticação (localStorage) e navegação
│   │   ├── index.css                # estilos globais
│   │   ├── components/              # "casca" da aplicação: Header, Body, Profile, Configs (.jsx + .css)
│   │   └── auth/                    # tela de login/cadastro: Login.jsx, Login.css
│   ├── index.html
│   ├── vite.config.js
│   ├── eslint.config.js
│   └── package.json
├── backend/                         # backend em Java (Spring Boot + Maven)
│   ├── pom.xml                      # dependências e build
│   ├── application-local.properties.example  # modelo da config local do banco (copie sem o .example)
│   ├── mvnw, mvnw.cmd, .mvn/        # Maven Wrapper (não precisa instalar o Maven)
│   └── src/
│       ├── main/java/br/com/marketfaesa/
│       │   ├── MarketFaesaApplication.java  # ponto de entrada (@SpringBootApplication)
│       │   ├── model/               # entidades JPA: Usuario (login por e-mail), ConfiguracaoUsuario, Tema
│       │   ├── controller/
│       │   ├── service/
│       │   └── repository/          # UsuarioRepository, ConfiguracaoUsuarioRepository (Spring Data JPA)
│       ├── main/resources/application.properties  # configurações (porta, banco etc.)
│       ├── main/resources/db/migration/           # migrations do Flyway (V1__..., V2__...)
│       └── test/java/br/com/marketfaesa/          # testes (JUnit + Spring Boot Test)
│           └── PostgresDeTeste.java               # banco PostgreSQL usado pelos testes
├── CONTRIBUTING.md                  # padrão de branches, commits e PRs
├── README.md
└── PROXIMOS_PASSOS.md
```

### Convenções do frontend

- **Organização por funcionalidade**: `components/` guarda a casca da aplicação (cabeçalho, corpo, perfil, configurações), e `auth/` tudo do login/cadastro.
- **CSS ao lado do componente**: cada `Componente.jsx` tem seu `Componente.css` na mesma pasta.
- **Nomes**: pastas em minúsculas; arquivos de componente em PascalCase (`Header.jsx`, `Login.jsx`); hooks em camelCase começando com `use`.
- **CSS global**: os estilos compartilhados são importados em `App.jsx` numa ordem fixa. A ordem importa para a cascata; ao adicionar um import, não reordene os existentes.
- **Assets**: arquivos de `public/` não são importados; são referenciados por caminho relativo (ex.: `./Imagens/Sino.svg`), para funcionar com o `base` do Vite.
- **Onde colocar código novo**:
  - nova tela ou parte da casca → `components/`; funcionalidade maior e independente → nova pasta em `src/` (ex.: `src/loja/`);
  - hooks e utilitários de uma funcionalidade → subpastas `hooks/` e `utils/` dentro dela;
  - chamadas HTTP (futuras) → `src/api/` (ver [PROXIMOS_PASSOS.md](PROXIMOS_PASSOS.md)).

### Fluxo de dados hoje

`App.jsx` concentra o estado (sessão, tema, configurações) e o repassa por props. Cada alteração é gravada no `localStorage` e lida de volta ao carregar a página; não existe servidor envolvido.

### Camadas do backend

- `model`: entidades JPA mapeadas para as tabelas das migrations (hoje `Usuario` e `ConfiguracaoUsuario`, com o enum `Tema`).
- `repository`: acesso e persistência dos dados (interfaces `JpaRepository`).
- `service`: regras de negócio e validações.
- `controller`: entrada das requisições; chama os services, sem regra de negócio.

## Como rodar o frontend

Pré-requisito: Node.js 20+ (a mesma versão usada no deploy).

```bash
cd MarketPlace
npm install
npm run dev       # servidor de desenvolvimento em http://localhost:5173/Projeto-Integrador-Computacional/
```

Outros scripts:

| Comando | O que faz |
|---|---|
| `npm run build` | gera a versão de produção em `MarketPlace/dist` |
| `npm run preview` | serve o build localmente |
| `npm run lint` | roda o ESLint |

## Como rodar o backend

Pré-requisito: JDK 21 ou superior (`java -version`). Não é preciso instalar o Maven: o Maven Wrapper (`mvnw`) baixa a versão certa na primeira execução.

O backend usa PostgreSQL hospedado no [Neon](https://neon.tech) (plano gratuito). Não há banco em memória nem Docker: tanto o desenvolvimento quanto a produção falam com um Postgres de verdade.

### Banco de dados (Neon)

O schema é criado só pelas migrations do Flyway em `backend/src/main/resources/db/migration/`, aplicadas automaticamente ao subir a aplicação. O Hibernate roda com `ddl-auto=validate`: ele só confere se as entidades batem com as tabelas e nunca altera o banco. Para mudar o schema, crie uma nova migration (`V<n>__descricao.sql`); nunca edite uma que já foi aplicada.

A conexão vem de variáveis de ambiente, lidas em `application.properties`:

| Variável | Uso |
|---|---|
| `DATABASE_URL` | URL JDBC do Neon **com pooler** (host com `-pooler`), usada pela aplicação |
| `DATABASE_USERNAME` | usuário do banco |
| `DATABASE_PASSWORD` | senha do banco |
| `DATABASE_DIRECT_URL` | opcional: URL JDBC **direta** (host sem `-pooler`), usada pelo Flyway nas migrations; se faltar, usa `DATABASE_URL` |

O Neon mostra a conexão como URI (`postgresql://usuario:senha@host/neondb?sslmode=require`). O JDBC não aceita esse formato: troque o prefixo por `jdbc:postgresql://`, tire `usuario:senha@` da URL e passe usuário e senha nas variáveis separadas. Exemplo:

```
DATABASE_URL=jdbc:postgresql://ep-xxxx-pooler.sa-east-1.aws.neon.tech/neondb?sslmode=require
```

### Rodando localmente

Em vez de exportar as variáveis, você pode usar o perfil `local`: copie `backend/application-local.properties.example` para `backend/application-local.properties` (já ignorado pelo Git, nunca faça commit dele), preencha com os dados do seu banco no Neon e rode, dentro de `backend/`:

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=local   # sobe o servidor em http://localhost:8080
```

Com as variáveis de ambiente definidas, basta `./mvnw spring-boot:run`.

No PowerShell ou no CMD, use `.\mvnw.cmd` no lugar de `./mvnw` (e `"-Dspring-boot.run.profiles=local"` entre aspas no PowerShell). O build vai para `backend/target/` (ignorada pelo `backend/.gitignore`).

### Testes

```bash
./mvnw test
```

Os testes não usam o Neon. A classe `src/test/java/br/com/marketfaesa/PostgresDeTeste.java` fornece o banco:

- se a variável `TEST_DATABASE_URL` existir (com `TEST_DATABASE_USERNAME` e `TEST_DATABASE_PASSWORD` opcionais), usa esse banco;
- senão, sobe um PostgreSQL 16 embutido ([zonky embedded-postgres](https://github.com/zonkyio/embedded-postgres)), baixado como dependência do Maven, sem instalar nada.

O Flyway aplica as migrations nesse banco antes dos testes. Se o Postgres embutido não subir no seu ambiente (ex.: sem permissão para extrair e executar os binários em `/tmp`), ou se preferir usar um Postgres já instalado, aponte `TEST_DATABASE_URL` para ele:

```bash
TEST_DATABASE_URL=jdbc:postgresql://localhost:5432/mf_test TEST_DATABASE_USERNAME=mf TEST_DATABASE_PASSWORD=mf ./mvnw test
```

Como escrever testes que usam o banco:

- **`@SpringBootTest`**: não precisa fazer nada. `PostgresDeTeste` é um `@Configuration` no pacote `br.com.marketfaesa`, então o component scan o encontra e o `DataSource` dele (marcado com `@Primary` e `@FlywayDataSource`) substitui o do `application.properties`.
- **Testes de fatia** (ex.: `@DataJpaTest`, que não fazem component scan): adicione `@Import(PostgresDeTeste.class)` e `@AutoConfigureTestDatabase(replace = Replace.NONE)`.

Ainda não há endpoints: acessar `http://localhost:8080` responde `404` até o primeiro controller ser criado (ver [PROXIMOS_PASSOS.md](PROXIMOS_PASSOS.md)).

## Deploy

A cada push na branch `main`, o workflow `.github/workflows/deploy.yml` instala as dependências, roda `npm run build` em `MarketPlace/` e publica o `dist` no GitHub Pages. O `base` do Vite está configurado como `/Projeto-Integrador-Computacional/`; se o nome do repositório no Pages mudar, esse valor precisa ser atualizado em `MarketPlace/vite.config.js`.

O GitHub Pages só hospeda arquivos estáticos, então o backend vai precisar de hospedagem separada.

## Como contribuir

O padrão de branches, commits e pull requests está em [CONTRIBUTING.md](CONTRIBUTING.md).

## Próximos passos

O roadmap do backend e da integração com o front está em [PROXIMOS_PASSOS.md](PROXIMOS_PASSOS.md).
