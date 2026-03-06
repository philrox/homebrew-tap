# Homebrew Tap Setup für ris-cli

## Ziel

Erstelle das GitHub-Repository `philrox/homebrew-tap` und konfiguriere alles, damit GoReleaser bei jedem Tag-Push automatisch die Homebrew-Formula aktualisiert.

## 1. Repository erstellen

Erstelle ein neues **öffentliches** GitHub-Repository:

- **Owner:** philrox
- **Name:** `homebrew-tap`
- **Beschreibung:** "Homebrew formulae for philrox CLI tools"
- **Öffentlich** (muss öffentlich sein, damit `brew install` funktioniert)
- **Mit README initialisieren**

Konvention: Homebrew Taps folgen dem Namensschema `homebrew-tap`. Nutzer installieren dann mit:

```bash
brew install philrox/tap/ris
```

(`homebrew-` Prefix wird automatisch weggelassen → `philrox/tap`)

## 2. Personal Access Token (PAT) erstellen

Der Standard-`GITHUB_TOKEN` in GitHub Actions hat nur Schreibrechte auf das eigene Repository (`ris-cli`). GoReleaser muss aber in ein **anderes** Repository (`homebrew-tap`) pushen. Dafür braucht es einen PAT.

### Fine-grained PAT erstellen (empfohlen)

1. Gehe zu: https://github.com/settings/personal-access-tokens/new
2. Einstellungen:
   - **Token name:** `goreleaser-homebrew-tap`
   - **Expiration:** 90 Tage (oder länger, nach Bedarf)
   - **Resource owner:** `philrox`
   - **Repository access:** "Only select repositories" → `philrox/homebrew-tap`
   - **Permissions:**
     - Repository permissions → **Contents:** Read and write
3. Token generieren und kopieren

### Alternativ: Classic PAT

1. Gehe zu: https://github.com/settings/tokens/new
2. Einstellungen:
   - **Note:** `goreleaser-homebrew-tap`
   - **Scope:** `repo` (Full control of private repositories)
3. Token generieren und kopieren

## 3. Secret in ris-cli Repository konfigurieren

1. Gehe zu: https://github.com/philrox/ris-cli/settings/secrets/actions
2. Klicke auf **"New repository secret"**
3. Einstellungen:
   - **Name:** `HOMEBREW_TAP_TOKEN`
   - **Value:** Den kopierten PAT einfügen
4. Speichern

## 4. Release-Workflow anpassen

Die Datei `.github/workflows/release.yml` muss den neuen Token verwenden. Ändere die `GITHUB_TOKEN`-Zeile:

```yaml
env:
  GITHUB_TOKEN: ${{ secrets.HOMEBREW_TAP_TOKEN }}
```

**Wichtig:** `HOMEBREW_TAP_TOKEN` wird sowohl für das GitHub Release als auch für den Homebrew-Push verwendet. Ein PAT mit `repo`-Scope deckt beides ab.

### Alternativ: Separaten Token nur für Homebrew

Falls du den Standard-`GITHUB_TOKEN` für Releases behalten willst und den PAT nur für Homebrew verwenden möchtest, passe stattdessen `.goreleaser.yml` an:

```yaml
brews:
  - repository:
      owner: philrox
      name: homebrew-tap
      token: "{{ .Env.HOMEBREW_TAP_TOKEN }}"
```

Und in `release.yml` beide Tokens setzen:

```yaml
env:
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  HOMEBREW_TAP_TOKEN: ${{ secrets.HOMEBREW_TAP_TOKEN }}
```

Diese Variante ist sauberer, weil der Standard-Token für Releases reicht und der PAT nur minimal verwendet wird.

## 5. Erster Release testen

```bash
git tag v0.1.0
git push origin v0.1.0
```

### Verifizieren

1. **GitHub Release:** https://github.com/philrox/ris-cli/releases → Release mit Binaries vorhanden?
2. **Homebrew-Tap:** https://github.com/philrox/homebrew-tap → `Formula/ris.rb` wurde erstellt?
3. **Installation testen:**

```bash
brew install philrox/tap/ris
ris version
```

## Checkliste

- [ ] Repository `philrox/homebrew-tap` erstellt (öffentlich)
- [ ] Personal Access Token erstellt mit Contents-Schreibrechten auf `homebrew-tap`
- [ ] Secret `HOMEBREW_TAP_TOKEN` in `ris-cli` Repository hinterlegt
- [ ] `release.yml` bzw. `.goreleaser.yml` angepasst (Token-Referenz)
- [ ] Erster Tag gepusht und Release verifiziert
- [ ] `brew install philrox/tap/ris` funktioniert
