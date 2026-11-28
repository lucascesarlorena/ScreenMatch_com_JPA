# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Screenmatch: a console (non-web) Spring Boot 3.1 / Java 17 app from Alura's "Java: persistência de dados e consultas com Spring Data JPA" course. It searches TV series on the OMDb API, translates synopses to Portuguese, and persists series in PostgreSQL via Spring Data JPA. Code, identifiers, and commit messages are in Portuguese — keep that convention.

## Commands

Use the Maven wrapper (`mvnw.cmd` on Windows, `./mvnw` in Git Bash):

- Build: `./mvnw clean package` (add `-DskipTests` to skip the context test)
- Run: `./mvnw spring-boot:run`
- All tests: `./mvnw test`
- Single test: `./mvnw test -Dtest=ScreenmatchApplicationTests#contextLoads`

Required environment variables (read by `src/main/resources/application.properties`): `DB_HOST` (host[:port]), `DB_NAME`, `DB_USER`, `DB_PASSWORD`. Schema is managed by Hibernate with `ddl-auto=update` — there are no migrations.

The only test is a `@SpringBootTest` context-load test. It boots the full app, so it needs a reachable database and also triggers the `CommandLineRunner` interactive menu, which reads from `System.in`.

## Architecture

- **Entry point**: `ScreenmatchApplication` implements `CommandLineRunner`; it gets `SerieRepository` injected and hands it to `Principal` (constructed manually with `new`, not a Spring bean). `Principal.exibeMenu()` is a `Scanner`-driven loop and is where all user-facing features live.
- **Two-layer model**:
  - `Dados*` records (`DadosSerie`, `DadosTemporada`, `DadosEpisodio`, `DadosTraducao`, `DadosResposta`) are Jackson DTOs mapping raw API JSON via `@JsonAlias` + `@JsonIgnoreProperties(ignoreUnknown = true)`.
  - `Serie` and `Episodio` are JPA entities (tables `series`, `episodios`) built from those records through constructors that do the cleanup: parsing ratings/dates with fallbacks, mapping OMDb's first genre to the `Categoria` enum via `Categoria.fromString` (throws on unknown genres), and translating the synopsis.
- **Services** (plain classes, not Spring beans, instantiated with `new`): `ConsumoApi` does HTTP GETs with `java.net.http.HttpClient`; `ConverteDados` (`IConverteDados`) deserializes JSON into a given class.
- **Translation**: `Serie`'s constructor calls `ConsultaMyMemory.obeterTraducao` (free MyMemory API, en→pt-br) on every new series — a network call per construction. `ConsultaChatGPT` (OpenAI completions via `openai-gpt3-java`) is an alternative left commented out; it needs a real key in place of its placeholder string.
- **Persistence**: `SerieRepository` extends `JpaRepository<Serie, Long>` with no custom queries yet. `Serie.titulo` is unique, so saving the same series twice fails. `Serie` ↔ `Episodio` is `@OneToMany(mappedBy = "serie")` / `@ManyToOne` with no cascade; episodes are currently only fetched and printed (`buscarEpisodioPorSerie`), not persisted. `Episodio` has no no-arg constructor, which JPA will need once episodes are loaded from the database.
- The OMDb base URL and API key are hard-coded constants in `Principal`.
