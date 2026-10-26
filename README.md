# Migrenownik na Androida

Aplikacja na Androida z powiadomieniami:
- **2 h po „Boli mnie teraz”** przychodzi powiadomienie, żeby uzupełnić natężenie, leki albo zakończyć ból (czas zmienisz w Ustawieniach: 1, 2, 3 h albo wcale),
- **codziennie o 21:00** przypomnienie o check-inie (do zmiany albo wyłączenia w Ustawieniach).

Powiadomienia działają bez internetu i bez serwera. Dane zostają tylko na telefonie.

Plik APK buduje się za darmo na GitHubie, więc nie musisz niczego instalować na komputerze.

## 1. Wgraj projekt na GitHub (raz, ok. 10 minut)

1. Załóż darmowe konto na github.com i zaloguj się.
2. Kliknij **+** w prawym górnym rogu, potem **New repository**. Nazwa: `migrenownik`. Może być **Private**. Kliknij **Create repository**.
3. Na stronie nowego repozytorium kliknij link **uploading an existing file**.
4. Rozpakuj paczkę z projektem i przeciągnij do okna przeglądarki **wszystkie pliki i foldery** z jej środka: `www`, `assets`, `android-res`, `package.json`, `package-lock.json`, `capacitor.config.json`, `README.md`, `.gitignore`. Kliknij **Commit changes**.
5. Teraz dodaj plik, który buduje aplikację. Foldery z kropką na początku często nie przechodzą przy przeciąganiu, więc zrób to ręcznie:
   - kliknij **Add file**, potem **Create new file**,
   - w polu nazwy wpisz dokładnie: `.github/workflows/build-apk.yml`,
   - wklej całą treść z sekcji **Plik budujący** na dole tej instrukcji,
   - kliknij **Commit changes**.

## 2. Pobierz gotową aplikację (za każdym razem ok. 5–8 minut)

1. W repozytorium otwórz zakładkę **Actions**. Zobaczysz zadanie **Zbuduj APK**. Uruchamia się samo po każdej zmianie plików.
2. Poczekaj, aż kółko zmieni się w zielony znaczek.
3. Kliknij zadanie, przewiń w dół do **Artifacts** i pobierz **Migrenownik-apk**. To plik .zip, w środku jest **Migrenownik.apk**.

Najprościej zrobić to od razu na telefonie z Androidem: zaloguj się na github.com w przeglądarce telefonu i pobierz plik stamtąd.

## 3. Zainstaluj na telefonie

1. Rozpakuj zip i otwórz **Migrenownik.apk**.
2. Android zapyta o zgodę na instalację z tego źródła. Pozwól (to jednorazowe ustawienie dla przeglądarki lub menedżera plików).
3. Otwórz Migrenownika i zezwól na powiadomienia.
4. Wejdź w **Ustawienia → Powiadomienia → Wyślij próbne powiadomienie za 5 s**, żeby sprawdzić, czy działa.

**Ważne na telefonach Samsung, Xiaomi, Huawei, OnePlus:** te telefony potrafią wstrzymywać przypomnienia, żeby oszczędzać baterię. Wejdź w ustawienia telefonu → Aplikacje → Migrenownik → Bateria i wybierz **Bez ograniczeń**.

## Aktualizacje

Gdy zmienisz pliki (np. podmienisz `www/index.html` na nową wersję), GitHub zbuduje nowy APK. Zainstaluj go na starą wersję. Dane zostaną, bo każda wersja jest podpisana tym samym kluczem z folderu `android-res`. Nie usuwaj tego pliku.

Odinstalowanie aplikacji usuwa wszystkie wpisy.

## Plik budujący

Treść pliku `.github/workflows/build-apk.yml`:

```yaml
name: Zbuduj APK

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21

      - name: Zainstaluj zależności
        run: npm ci

      - name: Przygotuj projekt Androida
        run: |
          npx cap add android
          npx capacitor-assets generate --android --iconBackgroundColor '#060508' --iconBackgroundColorDark '#060508' --splashBackgroundColor '#060508' --splashBackgroundColorDark '#060508'
          cp android-res/drawable/ic_stat_migrenownik.xml android/app/src/main/res/drawable/
          npx cap sync android

      - name: Stały klucz podpisu (aktualizacje bez utraty danych)
        run: |
          mkdir -p ~/.android
          cp android-res/debug.keystore ~/.android/debug.keystore

      - name: Zbuduj APK
        working-directory: android
        run: |
          chmod +x gradlew
          ./gradlew assembleDebug --no-daemon

      - name: Zmień nazwę pliku
        run: cp android/app/build/outputs/apk/debug/app-debug.apk Migrenownik.apk

      - uses: actions/upload-artifact@v4
        with:
          name: Migrenownik-apk
          path: Migrenownik.apk
```
