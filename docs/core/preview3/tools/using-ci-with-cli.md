---
title: "Utilisation du SDK et des outils .NET Core avec l’intégration continue │ Microsoft Docs"
description: "Utilisation du SDK et des outils .NET Core avec l’intégration continue"
keywords: .NET, .NET Core
author: blackdwarf
ms.author: mairaw
ms.date: 06/20/2016
ms.topic: article
ms.prod: .net-core
ms.technology: dotnet-cli
ms.devlang: dotnet
ms.assetid: 0d6e1e34-277c-4aaf-9880-3ebf81023857
translationtype: Human Translation
ms.sourcegitcommit: 796df1549a7553aa93158598d62338c02d4df73e
ms.openlocfilehash: 95c7f0f9911c7cb37c12afec74d0e942db77fbf6

---

# <a name="using-net-core-sdk-and-tools-in-continuous-integration-ci-net-core-tools-rc4"></a>Utilisation du SDK et des outils .NET Core avec l’intégration continue (outils .NET Core RC4)

> [!WARNING]
> Cette rubrique s’applique aux outils .NET Core RC4. Pour la version Preview 2 des outils .NET Core, consultez la rubrique [Utilisation du SDK et des outils .NET Core avec l’intégration continue](../../tools/using-ci-with-cli.md).

## <a name="overview"></a>Vue d'ensemble
Ce document décrit l’utilisation du SDK .NET Core et de ses outils sur le serveur de builds. En général, sur un serveur de builds avec intégration continue, il est souhaitable d’automatiser l’installation. Dans l’idéal, l’automatisation n’a pas besoin de privilèges d’administrateur. 

Pour les solutions SaaS avec intégration continue, il existe plusieurs options. Ce document aborde deux solutions SaaS très populaires, [TravisCI](https://travis-ci.org/) et [AppVeyor](https://www.appveyor.com/). Il existe, bien sûr, beaucoup d’autres services, mais les mécanismes d’installation et d’utilisation sont probablement similaires.

## <a name="installation-options-for-ci-build-servers"></a>Options d’installation pour les serveurs de builds avec intégration continue

## <a name="using-the-native-installers"></a>Utilisation de programmes d’installation natifs
Si l’utilisation de programmes d’installation nécessitant des privilèges d’administration ne pose pas problème, les programmes d’installation natifs de chaque plateforme peuvent être utilisés pour configurer le serveur de builds. Cette approche présente l’avantage d’installer automatiquement les dépendances nécessaires à l’exécution du SDK, ce qui est particulièrement intéressant pour les serveurs de builds Linux. Les programmes d’installation natifs installent également une version du SDK sur l’ensemble du système, ce qui peut vous être utile. Si ce n’est pas le cas, lisez la section ci-dessous concernant l’[utilisation du script d’installation](#using-the-installer-script). 

L’utilisation de cette méthode est simple. Pour Linux, vous pouvez utiliser un gestionnaire de package basé sur les flux RSS, tel que `apt-get` pour Ubuntu ou `yum` pour CentOS, ou utiliser directement les packages (c’est-à-dire, DEB ou RPM). La première option nécessite la configuration du flux qui contient les packages.

Pour les plateformes Windows, vous pouvez utiliser le fichier MSI. 

Tous les fichiers binaires se trouvent sur la [page de prise en main de .NET Core](https://aka.ms/dotnetcoregs) qui indique les versions stables les plus récentes. Si vous souhaitez utiliser des versions plus récentes (et potentiellement instables) ou la toute dernière version, vous pouvez cliquer sur les liens du [dépôt sur les outils CLI](https://github.com/dotnet/cli). 

## <a name="using-the-installer-script"></a>Utilisation du script d’installation
L’utilisation du script d’installation permet une installation non administrative sur votre serveur de builds. Elle facilite également l’automatisation. Le script va télécharger les fichiers ZIP/TAR nécessaires et les décompresser. Il va également ajouter l’emplacement d’installation sur l’ordinateur local au chemin (PATH) pour que les outils puissent être appelés immédiatement après l’installation. 

Le script d’installation peut être facilement automatisé au début de la génération pour récupérer et installer la version nécessaire du SDK. La « version nécessaire » est celle dont a besoin l’application actuellement développée. Vous pouvez choisir le chemin d’installation afin de pouvoir installer le SDK localement, puis nettoyer une fois la génération terminée. Cela apporte une encapsulation et une atomicité supplémentaires au processus de génération. 

La documentation de référence sur le script d’installation est disponible dans [dotnet-install](dotnet-install-script.md). 

### <a name="dealing-with-the-dependencies"></a>Gestion des dépendances
Si vous utilisez le script d’installation, cela signifie que les dépendances natives ne sont pas installées automatiquement et que vous devez les installer si elles ne sont pas déjà présentes sur le système d’exploitation. La liste des prérequis se trouve dans le [dépôt sur les outils CLI](https://github.com/dotnet/core/blob/master/Documentation/prereqs.md). 

## <a name="ci-services-setup-examples"></a>Exemples de configuration des services d’intégration continue
Les sections suivantes donnent des exemples de configurations utilisant les solutions SaaS d’intégration continue mentionnées précédemment. 

### <a name="travisci"></a>TravisCI

[travis-ci](https://travis-ci.org/) peut être configuré pour installer le SDK .NET Core à l’aide du langage `csharp` et de la clé `dotnet`.

Il vous suffit d’utiliser :

```yaml
dotnet: 1.0.0-preview2-003121
```

Travis peut exécuter aussi bien une tâche `osx` (OS X 10.11) qu’une tâche `linux` (Ubuntu 14.04) dans une matrice de génération. Pour plus d’informations, consultez cet [exemple .travis.yml](https://github.com/dotnet/docs/blob/master/.travis.yml).

### <a name="appveyor"></a>AppVeyor

Le SDK .NET Core Preview 2 est déjà installé dans l’image du travail de génération `Visual Studio 2015` de l’[appveyor.com-ci](https://www.appveyor.com/).

Il vous suffit d’utiliser :

```yaml
os: Visual Studio 2015
```

Il est possible d’installer une version spécifique du SDK .NET Core. Pour plus d’informations, consultez cet [exemple appveyor.yml](https://github.com/dotnet/docs/blob/master/appveyor.yml). 

Dans l’exemple, les fichiers binaires du SDK .NET Core sont téléchargés, décompressés dans un sous-répertoire, puis ajoutés au `PATH` env-var.

Une matrice de génération peut être ajoutée pour exécuter des tests d’intégration avec plusieurs versions du SDK .NET Core.

```yaml
environment:
  matrix:
    - CLI_VERSION: 1.0.0-preview2-003121
    - CLI_VERSION: Latest

install:
  # .NET Core SDK binaries
  - ps: $url = "https://dotnetcli.blob.core.windows.net/dotnet/preview/Binaries/$($env:CLI_VERSION)/dotnet-dev-win-x64.$($env:CLI_VERSION.ToLower()).zip"
  # follow normal installation from binaries
```


<!--HONumber=Feb17_HO2-->

