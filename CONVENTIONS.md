# Conventions de packaging des mods Per Aspera

> Format consommé par le **PerAspera Mod Launcher** et attendu pour toute entrée
> de `registry.json`. Un mod conforme s'installe par simple extraction du zip à
> la racine du jeu — le launcher n'a aucune logique par-mod.

## Règle unique : la racine du ZIP = la racine du jeu

Chaque release GitHub d'un mod contient **un seul asset ZIP** dont l'arborescence
reproduit celle de l'installation du jeu (`...\steamapps\common\Per Aspera\`).

### Mod YAML

```
WaterWork-v1.1.0.zip
└── Per Aspera_Data/
    └── StreamingAssets/
        └── Mods/
            └── WaterWork/          ← <id> du registry = nom du dossier
                ├── manifest.yaml
                ├── building.yaml
                └── Sprite/...
```

### Mod C# (plugin BepInEx)

```
WaterWorkFix-v1.0.1.zip
└── BepInEx/
    └── plugins/
        └── WaterWorkFix/           ← <id> du registry = nom du dossier
            └── WaterWorkFix.dll
```

### Mod mixte (YAML + C#)

Un seul zip combinant les deux arborescences ci-dessus.

## Versionnage

- Tags git `vX.Y.Z` (semver). Le nom de l'asset : `<id>-vX.Y.Z.zip`.
- Une release GitHub = une version stable installable. Pas de pre-releases dans
  le registry (le launcher ne montre que la latest release).

## Publication automatisée

Utiliser le workflow réutilisable de l'org — caller minimal dans le repo du mod
(`.github/workflows/release.yml`) :

```yaml
name: Release
on:
  push:
    tags: ['v*']
permissions:
  contents: write
jobs:
  release:
    uses: PerAsperaMods/.github/.github/workflows/release-mod.yml@main
    with:
      mod-id: MonMod
      mod-type: yaml        # ou csharp
    secrets:
      gamelibs-token: ${{ secrets.GAMELIBS_TOKEN }}   # csharp uniquement
```

- **yaml** : le contenu du repo (moins `.git*`, `.github/`, `README.md`, `LICENSE*`)
  est stagé sous `Per Aspera_Data/StreamingAssets/Mods/<mod-id>/` puis zippé.
- **csharp** : build contre le modder-pack de la dernière release SDK + interop du
  jeu (repo privé, secret requis), la DLL produite est zippée sous
  `BepInEx/plugins/<mod-id>/`. Le `.csproj` ne doit pas dépendre de chemins
  locaux quand `GITHUB_ACTIONS=true` (conditionner les `ProjectReference`).

## Entrée registry.json

```json
{
  "id": "MonMod",              // nom du dossier d'install (= mod-id du workflow)
  "name": "Mon Mod",
  "description": "…",
  "type": "yaml | csharp | mixed",
  "repo": "PerAsperaMods/MonMod",
  "sdkMinVersion": "1.2.0",    // null si aucun besoin du SDK
  "gameVersion": "1.8.x",
  "dependencies": ["AutreModId"]   // mods du registry requis
}
```

Pour ajouter un mod au registry : PR sur ce repo modifiant `registry.json`.
