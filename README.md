# 🎬 ScreenMatch

Aplicação para buscar séries de TV na [OMDb API](https://www.omdbapi.com/), traduzir as sinopses para português e guardar tudo num banco PostgreSQL com Spring Data JPA. Os dados ficam disponíveis numa API REST, consumida por um front-end em HTML/CSS/JS.

Projeto desenvolvido durante os cursos da formação **Avançando com Java** da Alura:

- *Java: persistência de dados e consultas com Spring Data JPA*
- *Java: criando uma aplicação web com Spring*

## 🛠️ Tecnologias

| Camada     | Tecnologias |
|------------|-------------|
| Back-end   | Java 17, Spring Boot 3.1, Spring Web, Spring Data JPA, Hibernate, Jackson |
| Banco      | PostgreSQL |
| APIs externas | OMDb (dados das séries), MyMemory (tradução das sinopses) |
| Front-end  | HTML, CSS e JavaScript puro |

## 📁 Estrutura

```
ScreenMatch_com_JPA/
├── backend/              # API Spring Boot
│   └── src/main/java/br/com/alura/screenmatch/
│       ├── config/       # Configuração de CORS
│       ├── controller/   # SerieController (rotas REST)
│       ├── dto/          # SerieDTO, EpisodioDTO
│       ├── model/        # Entidades JPA (Serie, Episodio), records Dados* e enum Categoria
│       ├── principal/    # Menu de console (Principal)
│       ├── repository/   # SerieRepository (derived queries e JPQL)
│       └── service/      # SerieService, consumo de APIs e conversão de JSON
└── frontend/             # Páginas que consomem a API
```

## ✅ Pré-requisitos

- JDK 17+
- PostgreSQL rodando
- (Opcional) VS Code com a extensão **Live Server** para abrir o front-end

## ▶️ Como rodar

### 1. Banco de dados

Crie um banco vazio no PostgreSQL. As tabelas (`series` e `episodios`) são criadas automaticamente pelo Hibernate (`ddl-auto=update`).

### 2. Variáveis de ambiente

O `application.properties` lê a conexão destas variáveis:

| Variável      | Exemplo           |
|---------------|-------------------|
| `DB_HOST`     | `localhost:5432`  |
| `DB_NAME`     | `screenmatch`     |
| `DB_USER`     | `postgres`        |
| `DB_PASSWORD` | `sua_senha`       |

PowerShell:

```powershell
$env:DB_HOST="localhost:5432"; $env:DB_NAME="screenmatch"; $env:DB_USER="postgres"; $env:DB_PASSWORD="sua_senha"
```

Bash:

```bash
export DB_HOST=localhost:5432 DB_NAME=screenmatch DB_USER=postgres DB_PASSWORD=sua_senha
```

### 3. Back-end

```bash
cd backend
./mvnw spring-boot:run      # Windows: mvnw.cmd spring-boot:run
```

A API sobe em `http://localhost:8080`.

### 4. Front-end

Abra a pasta `frontend` no VS Code, clique com o botão direito em `index.html` e escolha **Open with Live Server**.

> O CORS do back-end libera apenas `http://127.0.0.1:5501`. Se o Live Server abrir em outra porta, ajuste `config/CorsConfiguration.java`.

## 🔌 Endpoints

| Método | Rota                               | Descrição |
|--------|------------------------------------|-----------|
| GET    | `/series`                          | Todas as séries cadastradas |
| GET    | `/series/top5`                     | As 5 séries com maior avaliação |
| GET    | `/series/lancamentos`              | Séries com os episódios lançados mais recentemente |
| GET    | `/series/{id}`                     | Detalhes de uma série |
| GET    | `/series/{id}/temporadas/todas`    | Todos os episódios da série |
| GET    | `/series/{id}/temporadas/{numero}` | Episódios de uma temporada |
| GET    | `/series/categoria/{genero}`       | Séries de um gênero (nome em português, ex.: `Comédia`, `Drama`) |

Gêneros disponíveis: Ação, Romance, Comédia, Crime, Drama, Fantasia, Ficção Científica, Terror, Animação, Aventura e Documentário.

## 💻 Modo console

Antes da API, o projeto era uma aplicação de linha de comando com um menu interativo (classe `Principal`), que permite:

1. Buscar séries na OMDb e salvar no banco
2. Buscar e salvar os episódios de uma série
3. Listar as séries salvas
4. Buscar série por título
5. Buscar série por ator e avaliação mínima
6. Ver o top 5 de séries
7. Buscar séries por gênero
8. Filtrar séries por número de temporadas e avaliação
9. Buscar episódio por trecho do título
10. Ver o top 5 de episódios de uma série
11. Buscar episódios a partir de um ano

Esse modo é o cadastro de séries no banco: é por ele que os dados entram. Para usá-lo, descomente a classe `ScreenmatchApplicationSemWeb` e rode a aplicação por ela.

## 🎯 O que foi praticado

- Consumo de APIs externas com `HttpClient` e desserialização de JSON com Jackson
- Modelagem com classes, records e enums
- Mapeamento de entidades e relacionamentos `@OneToMany` / `@ManyToOne` com JPA
- Consultas com derived queries e JPQL (`@Query`) no `JpaRepository`
- Criação de uma API REST com Spring Web, DTOs e camada de serviço
- Configuração de CORS para integrar com o front-end
