---
title: "Jekyll na GitHub Pages — jak opublikować stronę jednym push"
date: 2026-05-14 12:00:00 +0000
categories: [blog, jekyll, hosting]
tags: [jekyll, github-pages, deployment, tutorial, chirpy]
---

W poprzednim wpisie obiecałem pokazać wdrożenie bloga na **GitHub Pages**. Poniżej masz praktyczny przewodnik: od repozytorium do działającej strony, z uwagami, które zwykle oszczędzają godziny debugowania.

## Co dokładnie robi GitHub Pages z Jekyll?

GitHub może zbudować statyczną stronę z Twojego repozytorium na dwa główne sposoby:

1. **GitHub Actions** (zalecane) — workflow buduje projekt (np. `bundle exec jekyll build`) i publikuje wynik z folderu `_site` (lub innego, który wskażesz).
2. **Klasyczny silnik Jekyll na serwerach GitHub** — działa tylko z [ograniczonym zestawem wtyczek i gemów](https://pages.github.com/versions/). Motywy takie jak **Chirpy** często wymagają Actions, bo przekraczają te limity.

Jeśli używasz motywu z `Gemfile` i nowszymi zależnościami, traktuj **Actions jako domyślną ścieżkę**.

## Repozytorium: strona użytkownika vs projekt

| Typ | Nazwa repozytorium | Adres (przykład) |
|-----|-------------------|------------------|
| Strona użytkownika / organizacji | `username.github.io` | `https://username.github.io/` |
| Strona projektu | dowolna, np. `moj-blog` | `https://username.github.io/moj-blog/` |

Dla strony **projektu** w `_config.yml` ustaw:

- `url` — pełny adres hosta Pages, np. `https://username.github.io`
- `baseurl` — ścieżka podfolderu, np. `/moj-blog` (bez końcowego slasha)

Dzięki temu linki do CSS, obrazków i wpisów nie będą prowadziły w próżnię.

## Szybka checklista przed pierwszym deployem

- [ ] Repozytorium **publiczne** (darmowy Pages) albo płatny plan z Pages dla prywatnego.
- [ ] W gałęzi domyślnej jest kod źródłowy Jekyll (`_config.yml`, `Gemfile`, itd.).
- [ ] W **Settings → Pages** wskazane jest źródło publikacji (np. gałąź `gh-pages` zbudowana przez Actions albo „GitHub Actions” jako źródło).
- [ ] Workflow Actions ma uprawnienia do zapisu (w repozytorium: **Settings → Actions → General → Workflow permissions**: *Read and write*).

## Typowy przepływ z GitHub Actions

1. W repozytorium masz plik workflow, np. `.github/workflows/pages-deploy.yml` (w szablonach Chirpy często jest już gotowiec).
2. Po `git push` do gałęzi źródłowej workflow:
   - instaluje Ruby i zależności z `Gemfile`,
   - uruchamia `jekyll build`,
   - wrzuca artefakt do gałęzi docelowej lub do mechanizmu „GitHub Pages artifact”.
3. Po kilku minutach strona jest pod adresem z ustawień **Pages**.

Pierwszy build bywa wolniejszy; kolejne są zwykle krótsze dzięki cache.

## Lokalnie: ten sam build co na Pages

Zanim wypchniesz zmiany:

```bash
bundle install
bundle exec jekyll serve --livereload
```

Sprawdź w przeglądarce, czy ścieżki zasobów działają — szczególnie przy **niepustym `baseurl`**. W razie potrzeby użyj filtra Liquid `relative_url` w szablonach (motyw Chirpy zwykle już to uwzględnia).

## Częste problemy

**Pusta strona albo brak stylów** — najczęściej `baseurl` / `url` niezgodne z rzeczywistym adresem repozytorium Pages.

**Build pada na GitHubie, lokalnie działa** — wersja Ruby albo gemów inna niż w Actions; ujednolić przez `.ruby-version` i `Gemfile.lock` z commitowanym lockiem.

**Wtyczka „nieobsługiwana” na klasycznym Pages** — przejść na workflow z Actions i pełnym `bundle exec jekyll build`.

## Domena własna (opcjonalnie)

W DNS ustawiasz rekordy według [dokumentacji GitHub](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site). W repozytorium dodajesz plik `CNAME` z nazwą hosta. W `_config.yml` aktualizujesz `url` na `https://twoja-domena.pl` i zwykle **czyścisz `baseurl`** dla strony w katalogu głównym domeny.

---

To wystarczy, żeby świadomie postawić Jekyll na GitHub Pages i uniknąć typowych pułapek z adresami i buildem. Jeśli chcesz, w kolejnym wpisie mogę rozwinąć sam plik workflow krok po kroku pod Twoje repozytorium.
