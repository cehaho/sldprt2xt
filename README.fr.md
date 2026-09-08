🇬🇧 [English version](README.md)

# sldprt2xt

Sort la géométrie d'un fichier SolidWorks. Sans SolidWorks.

`piece.SLDPRT` → `piece.x_t`, un fichier Parasolid.
## Installer

```sh
pip install sldprt2xt
```

## Utiliser

```sh
sldprt2xt piece.SLDPRT
```

Ça écrit `piece.x_t` juste à côté. C'est tout.

Ailleurs :

```sh
sldprt2xt piece.SLDPRT -o sorties/
```

Plusieurs fichiers :

```sh
sldprt2xt *.SLDPRT -o sorties/
```

Un dossier entier, sous-dossiers compris — l'arborescence est reproduite
dans la sortie, et les verrous SolidWorks (`~$pièce.SLDPRT`) sont ignorés :

```sh
sldprt2xt mes_pieces/ -o sorties/
```

Dans du Python :

```python
from sldprt2xt import to_x_t

to_x_t("piece.SLDPRT")              # écrit piece.x_t
to_x_t("piece.SLDPRT", "sorties/")  # ailleurs — le dossier est créé au besoin
```

Tout échec lève `ConversionError` (ou `SchemasNotFound`), avec un message qui
dit quoi faire. L'écriture est atomique : jamais de fichier tronqué sous son
nom final.

## Les tables sont dedans

Un fichier Parasolid ne se lit pas sans une table qui dit quels champs porte
chaque type de nœud. **Ce paquet énonce lui-même les faits de ces tables**
(`schema_facts.py`) : toutes les versions de SolidWorks connues à sa
publication — 2003 à 2026 — se convertissent sans rien d'autre.

Deux cas seulement où fournir quelque chose :

- **une version de SolidWorks plus récente que ce paquet**, pour une pièce à
  plusieurs corps. Trois sorties, du plus simple au plus manuel : mettre le
  paquet à jour ; donner un `.x_t` multi-corps exporté par le même
  SolidWorks (`--donor piece.x_t` — l'outil y apprend ce qui lui manque) ;
  ou pointer `--schemas` vers le dossier `pschema` d'une installation.
- **vos propres fichiers de schéma**, si vous préférez : un dossier déposé
  prime toujours sur les tables intégrées (`--schemas`, ou la variable
  `P_SCHEMA` que Parasolid lui-même consulte — une installation SolidWorks
  locale est d'ailleurs trouvée toute seule).

## Ce qui sort

Tous les corps de la pièce, avec leurs couleurs. La géométrie est **celle du
fichier, recopiée** — rien n'est reconstruit, rien n'est approché.


**Ce qui n'est *pas* transmis, et pourquoi ça compte.** Une partition
SolidWorks porte bien plus que de la géométrie : identifiants permanents de
chaque arête et de chaque face, recettes de corps, horodatages, identifiant de
la dernière fonction ayant modifié le corps. Tout cela n'a de sens que dans son
propre modèle, son export ne le transmet jamais — et **il refuse de relire un
transmit qui le porte**. Mesuré sur une pièce de 49 corps : 31 définitions
d'attributs écrites là où son export en garde 7, et le fichier ne s'ouvrait
pas. Depuis la 1.2.0 elles restent en arrière, et le résultat s'ouvre en pièce
multi-corps, couleurs intactes.

> **Si vous êtes en 1.1.0, mettez à jour.** Elle écrit des fichiers que
> SolidWorks refuse avec *« The data in this file may be invalid »*, et là où
> une version plus ancienne en ouvre un, c'est en assemblage de N composants
> plutôt qu'en pièce à N corps — une marque manquait aussi sur la racine.

## Qui ouvre le `.x_t`

Les logiciels bâtis sur le noyau Parasolid : SolidWorks, NX, Solid Edge,
Fusion 360, Onshape, Rhino, Plasticity…

**Pas FreeCAD** — ni rien de construit sur OpenCASCADE. Il n'existe aucun
lecteur Parasolid libre ; c'est précisément la raison d'être de cet outil.
Pour la chaîne libre, il faut du STEP, et produire du STEP est un autre
métier — reconstruire la géométrie dans un noyau, pas la recopier — que cet
outil ne fait volontairement pas.

Cet autre métier fait l'objet d'un travail à part, pas encore publié. Le jour
où il le sera, attendez-vous à ce qu'il dise corps par corps ce qu'il a réussi
— parce qu'une reconstruction n'est pas une transcription, et que des deux,
une seule se vérifie contre le fichier.

## Ça marche sur quoi

Les `.SLDPRT` de SolidWorks 2003 à 2026, les deux générations de format de
fichier. 

Linux, macOS, Windows. Python 3.10 ou plus.

## Licence

AGPL-3.0. Voir [LICENSE](LICENSE).

Parasolid est une marque de Siemens Industry Software Inc., SolidWorks une
marque de Dassault Systèmes. Ce projet n'est affilié ni à l'un ni à l'autre.
