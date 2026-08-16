# Upgrading Apple Identity Provider for Keycloak

Since breaking changes not only occur within Keycloak, but also in this extension, it is necessary to migrate data in rare cases after an extension upgrade.

### Upgrade from <1.14.0 to 1.14.0 or later
- The field p8-Key got dropped in favor of the field Client-Secret in the Admin-UI in version 1.14.0. Please paste the p8-Key into Client-Secret instead. Otherwise you will get the error `"invalid_client"` from Apple and login does not work.

## Islamify-Fork

Dieser Fork wird von EMAD gUG (Islamify) betrieben und ausschliesslich fuer
`auth.islamify.ai` gebaut. Aenderungen gegenueber `klausbetz/apple-identity-provider-keycloak`:

- `build.gradle`: `keycloakVersion` auf die tatsaechlich laufende Keycloak-Version
  gepinnt (aktuell `26.5.6`). Vor jedem Keycloak-Upgrade hier nachziehen, neu bauen
  und zuerst auf einer Testinstanz pruefen — nie zuerst auf der Produktion.
- `AppleIdentityProvider.parseUser()`: Die E-Mail aus Apples `user`-JSON wird jetzt
  auch dann uebernommen, wenn Apple keinen Namen mitschickt. Vorher lag die
  E-Mail-Auswertung innerhalb der `name`-Bedingung und ging in diesem Fall verloren.

Bauen (ohne lokale Java-Installation):

    docker run --rm -v "$PWD":/w -w /w gradle:8.5-jdk17 gradle --no-daemon clean build -x javadoc

Das Ergebnis liegt unter `build/libs/apple-identity-provider-<version>.jar` und gehoert
auf dem Server nach `/opt/islamify/staging/data/keycloak/providers/`.
