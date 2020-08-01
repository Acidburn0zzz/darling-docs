# Émulation d'appel système

Les appels système sont l'interface la plus critique de tous les logiciels de l'espace
utilisateur. Ils sont le moyen d'invoquer la fonctionnalité du noyau. Sans eux, le 
logiciel ne pas pouvoir communiquer avec l'utilisateur, accéder au système de fichiers 
ou établir les connexions de réseau.

Chaque noyau fournit un ensemble très spécifique d'appels système, même s'il y a un 
ensemble similaire d'API libc disponibles sur différents systèmes. 
Par exemple, le système L'ensemble d'appels diffère grandement entre Linux et FreeBSD.

MacOS utilise un noyau nommé XNU. Les appels système de XNU diffèrent grandement de 
ceux de Linux, c'est pourquoi l'émulation des appels système XNU est au cœur de Darling.

## Appels système XNU

Contrairement aux autres noyaux, XNU possède trois tables d'appels système distinctes:

1. ** Appels système BSD **, appelés via sysenter / syscall. Ces appels fréquemment
      ont un équivalent direct sur Linux, ce qui les rend plus faciles à implémenter.
2. ** Appels système Mach **, qui utilisent des numéros d'appel système négatifs. 
      Ceux-ci sont
      inexistants sous Linux et sont principalement implémentés dans le Linux chéri
      module noyau. Ces appels sont construits autour de [Mach ports] 
      (../ macos-specifics / mach-ports.md).
3. ** Appels système dépendant de la machine **, appelés via `int 0x82`. Il n'y a qu'un
      peu d'entre eux et ils sont principalement liés à [thread-local stockage]
      (../ threading / thread-local-storage.md).

## Introduction

    Darling émule les appels système en fournissant un 
    Bibliothèque `/ usr / lib / system / libsystem_kernel.dylib`, dont le code source 
    est situé dans `src / kernel`. 
    Même si certaines parties de l'émulation se trouvent dans Module de noyau de Darling 
    (situé dans `src / external / lkm`), l'émulation de Darling est basé sur l'espace 
    utilisateur.

C'est pourquoi `libsystem_kernel.dylib` (et aussi` dyld`, qui contient un build de cette 
bibliothèque) ne peut jamais être copié de macOS vers Darling.

L'émulation des appels système XNU directement sous Linux aurait quelques avantages, mais
n'est pas vraiment réalisable. Contrairement aux noyaux BSD, Linux ne prend pas en charge 
les émulation d'appel système et fusion d'un ensemble de patchs aussi vaste et intrusif sous 
Linux serait trop difficile. 
Exiger des utilisateurs de Darling de patcher leur kernels est également hors de question.

### Inconvénients de cette approche

* Impossibilité d'exécuter une copie complète de macOS sous Darling (nonobstant le
  légalité d'une telle entreprise), au moins les fichiers mentionnés ci-dessus doivent être
  différent.

* Incapacité d'exécuter le logiciel faisant des appels système directs. 
  Cela inclut certains anciens exécutables compressés avec UPX. Dans le passé, 
  les logiciels écrits en Go appartenaient également à ce groupe, mais ce n'est plus le cas. 
  Notez que : 
  
  [Apple ne fournit aucun support] (https://developer.apple.com/library/content/qa/qa1118/_index.html)
  pour passer des appels système directs 
  
  (ce qui est en fait très similaire à distribuer des exécutables liés statiquement décrits 
  dans l'article) en plus de ça, Apple modifie fréquemment la table des appels système, ce 
  qui explique grandement les souçis que l'ont rencontre avec chaque nouvelle mise-à-jours 
  vers une version version majeur de leurs systeme et les logiciels que vous pouvez vous 
  procurrer en temps que logiciels tiers. Cette methodes fort discutable toutes notre 
  reimplementations est donc dors et déja vouée à echouée un jours non prévisible et ce pour des 
  raisons que nous ne pouvons prévoire. 
  Mais cela n'entame en aucun cas notre motivation car au fil du temp nous comprennont de mieux 
  les différents principes sous-jacent relatifs aux noyaux systemes des os en questions ce qui 
  techniquement est trés instructifs et pratiquement nous allons de plus en plus vite pour 
  re-transcrire une modifications pour qu'elle ne conduise pas vers une erreures. C'est donc 
  pour de bonne raisons que nous vous conseillons de ne pas utiliser des versions dite long term supp.
  Mais plustot de suivres notre repos github officiels et d'apprendre à actualiser vos sources 
  directement depuis les corrections afin de coller au mieux dans le temps avec nous au fils de l'eau.

### Avantages de cette approche
  * Développement nettement plus facile.
  * Pas besoin de s'inquiéter de la fusion des correctifs dans Linux.
  * Il est plus facile pour les utilisateurs d'avoir le dernier code disponible, 
    il n'est pas nécessaire de l'exécuter le dernier noyau à avoir l'émulation 
    d'appel système la plus récente.

## La mise en oeuvre

TODO
