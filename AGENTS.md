# keycloak-apple-idp – Anleitung für KI-Werkzeuge

Gilt für Codex, Antigravity und Claude. Emirs allgemeine Regeln: ~/.claude/CLAUDE.md.

## Zweck

Keycloak-Erweiterung für „Mit Apple anmelden“ (Sign in with Apple). Apple weicht von OpenID Connect ab
(Rückruf per `POST`, kein `userinfo`-Endpunkt, Name/E-Mail nur beim ersten Login, Client-Secret als
signiertes JWT aus einem `.p8`-Schlüssel); diese Erweiterung gleicht das aus. Das Repository ist ein
**öffentlicher** Fork von `klausbetz/apple-identity-provider-keycloak` und wird von EMAD gUG (Islamify)
ausschließlich für `auth.islamify.ai` gebaut. Fork-Änderungen: siehe Abschnitt „Islamify-Fork“ in `UPGRADE.md`.

## Aufbau

- `src/main/java/at/klausbetz/provider/` – der Provider (Java 17):
  - `AppleIdentityProvider.java` – Login-Ablauf, Token-Tausch, `parseUser()` (Fork-Fix: E-Mail auch ohne Namen)
  - `AppleIdentityProviderEndpoint.java` – nimmt Apples `form_post`-Rückruf an
  - `AppleIdentityProviderFactory.java`, `AppleIdentityProviderConfig.java` – Registrierung und Admin-Felder
  - `Apple*Mapper.java` – Attribut-, Benutzername- und Session-Mapper
  - `TokenExchangeParams.java` – Parameter für den Token-Tausch
- `src/main/resources/META-INF/services/` – SPI-Registrierung (Factory und Mapper); muss zu den Klassennamen passen
- `src/test/java/...` – JUnit-5-Tests (`TokenExchangeParamsTest`)
- `build.gradle` – Version der Erweiterung (`version`) und gepinnte Keycloak-Version (`keycloakVersion`)
- `README.md` – Upstream-Doku (Installation, Kompatibilitätstabelle, Konfiguration, Token-Tausch)
- `UPGRADE.md` – Upgrade-Hinweise plus Islamify-Fork-Abschnitt mit Build-Befehl und Zielordner
- `docs/` – Docker-Beispiel und Anleitung für Token-Tausch
- `.github/workflows/` – CI, Release und JitPack (Upstream-Workflows)

## Bauen / Testen

Ohne lokale Java-Installation (so in `UPGRADE.md` dokumentiert):

    docker run --rm -v "$PWD":/w -w /w gradle:8.5-jdk17 gradle --no-daemon clean build -x javadoc

Mit lokalem JDK 17 über den Gradle-Wrapper (Gradle 8.0.2), wie in der CI:

    ./gradlew test
    ./gradlew jar

Ergebnis: `build/libs/apple-identity-provider-<version>.jar`. `build/` ist in `.gitignore`.

## Auslieferung / Betrieb

- Das JAR kommt in den `providers`-Ordner von Keycloak; danach braucht Keycloak einen Neustart bzw.
  einen frischen Container. Zielordner auf dem Islamify-Server: siehe `UPGRADE.md`.
- `auth.islamify.ai` ist ein Produktivsystem: nichts dorthin ausrollen, neu starten oder umkonfigurieren
  ohne ausdrückliche Freigabe von Emir. Immer zuerst auf einer Testinstanz prüfen.
- Apple-Zugangsdaten (Service ID, Team ID, Key ID, `.p8`-Schlüssel) werden in der Keycloak-Admin-Oberfläche
  gepflegt und gehören **nie** ins Repository – auch nicht in Tests, Beispiele oder Commits.

## Regeln & Fallstricke

- **Öffentliches Repository:** keine internen Hostnamen, Serverpfade, Zugangsdaten, Nutzerdaten oder
  Hinweise auf private Repos hinzufügen. Was hier steht, ist für alle lesbar.
- **Keycloak-Version gepinnt:** Die Erweiterung nutzt interne Keycloak-Schnittstellen und kann mit neuen
  Keycloak-Versionen brechen (siehe Kompatibilitätstabelle im `README.md`). `keycloakVersion` in
  `build.gradle` soll der laufenden Keycloak-Version entsprechen – vor jeder Arbeit prüfen, ob der Wert
  noch stimmt. Vor einem Keycloak-Upgrade: Wert nachziehen, neu bauen, auf einer Testinstanz prüfen.
- **Tags lösen Releases aus:** Jeder gepushte Tag startet `release.yml` (GitHub-Release mit JAR) und
  `jitpack-build.yml`. Keine Tags pushen, ohne dass ein Release gewollt ist.
- **Upstream-Code:** Paket- und Gruppenname bleiben `at.klausbetz`; eigene Änderungen klein halten und
  im Islamify-Fork-Abschnitt von `UPGRADE.md` festhalten, damit ein späterer Abgleich mit dem Upstream
  nachvollziehbar bleibt. `LICENSE` (Apache 2.0) nicht verändern.
- **Admin-Feld „Client Secret“:** Seit 1.14.0 steht dort der Inhalt des `.p8`-Schlüssels (das alte Feld
  „p8 Key“ entfällt), siehe `UPGRADE.md`.
- **Private-Relay-E-Mails:** Damit Nutzer mit versteckter Apple-Adresse Mails von Keycloak erhalten, muss
  die Absenderadresse im Apple-Entwicklerkonto freigegeben sein (siehe `README.md`).

## Wo mehr Wissen liegt

- `README.md` – Installation, Kompatibilität, Konfigurationsfelder, Token-Tausch
- `UPGRADE.md` – Upgrade-Schritte und Islamify-Fork-Hinweise
- `docs/README_docker_installation.md`, `docs/CONFIGURE_TOKEN_EXCHANGE.md`
- Upstream-Wiki (Apple-Entwicklerportal, Fehlersuche): https://github.com/klausbetz/apple-identity-provider-keycloak/wiki
