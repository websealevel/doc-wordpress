# Visual Studio Code : environnement *Wordpress standards* friendly

Quand on développe du PHP on aime bien respecter ses standards. Wordpress a défini ses propres [standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/), basés sur les PSR officiels.

Le but est donc de produire du code *propre* du point de vue des standards de notre framework PHP. Utile si l'on veut produire du code maintenable par d'autres développeurs ou si l'on veut publier un thème ou un plugin par exemple.

## PHP_CodeSniffer

[PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer) c'est deux scripts:

- `phpcs` qui detecte des violations de standards (que l'on pourra choisir)
- `phpcbf` qui corrige automatiquement les violations du standard (notamment le formattage du code source)

Ici on va s'en servir pour configurer le standard de [Wordpress](https://github.com/WordPress/WordPress-Coding-Standards/tree/develop/WordPress) dans vscode.

C'est fastidieux mais on fini par s'en sortir.

## 1 - Installer [Composer](https://getcomposer.org/)

~~~bash
sudo apt install composer
~~~

Remplacez `apt` par votre gestionnaire de paquet favori.

## 2 - Installer PHP_CodeSniffer et ses dépendances localement

On va l'installer localement dans chaque projet. Vous pouvez aussi le faire globalement mais je préfère ainsi. Il faudra donc repenser seulement à lancer cette commande dans un nouveau projet (la configuration dans le `settings.json` sera conservée de projet en projet)

~~~bash
composer require squizlabs/php_codesniffer
composer require phpcsstandards/phpcsutils:@alpha
composer require phpcsstandards/phpcsextra:@alpha
~~~

On installe ici localement `php_codesniffer` ainsi que des fonctions utils utilisées par ce dernier.

## 3 - Installer phpcs: extension phpcodesniffer pour vscode

Installer l'extension [phpcs](https://marketplace.visualstudio.com/items?itemName=ikappas.phpcs) depuis le marketplace de vscode


## 4 - Installer phpcbf: extension pour vscode

Installer l'extension [vscode-phpcbf](https://marketplace.visualstudio.com/items?itemName=persoderlind.vscode-phpcbf) pour indiquer à vscode où trouver le bin

## 5 - Télécharger le standard de wordpress

- cloner [dépôt des codings standards de wordpress](https://github.com/WordPress/WordPress-Coding-Standards/tree/develop/WordPress).
- renommer le en `wpc`, c'est plus simple
- copier `wpc` à la racine de votre projet

Il ne nous reste plus qu'à indiquer à `phpcs` où se trouve notre standard Wordpress. 

A la racine du projet

~~~bash
./vendor/bin/phpcs --config-set installed_paths wpcs
~~~

Ok, ça fait beaucoup mais nous sommes fin prêts. Il ne reste plus qu'à configurer les extensions vscode.

## Configuration des extensions

Rendez-vous dans votre `settings.json` pour paramètrer l'extension qui a pour domaine phpcs. Copiez coller la config suivante

~~~json

    //config phpcs
    "phpcs.executablePath": "vendor/bin/phpcs",
    "phpcs.standard": "Wordpress",

    //config phpcb
    "phpcbf.executablePath": "vendor/bin/phpcbf",
    "phpcbf.standard": "WordPress",
    "phpcbf.documentFormattingProvider": true,
    "phpcbf.onsave": true,
    "phpcbf.enable": true,

    //config php files formatted by phpcbf
    "[php]": {
        "editor.defaultFormatter": "persoderlind.vscode-phpcbf"
    },
~~~

Ici on indique à vscode

- le path pour les scripts `phpcs` et `phpcbf`-
- d'utiliser le standard de Wordpress-
- de formater le code en corrigeant automatiquement les violations d'indentation du standard à la sauvegarde du fichier source
- d'en faire notre formatter php par défaut

Essayez, si tout s'est bien passé vous devriez voir des warnings partout : )

Have fun.

## Ressources

- [Dépot Wordpress coding standards](https://github.com/WordPress/WordPress-Coding-Standards/tree/develop/WordPress)
- [Setting up WordPress Coding Standards in VS Code](https://www.edmundcwm.com/setting-up-wordpress-coding-standards-in-vs-code/)
- [Setting Up PHP CodeSniffer in Visual Studio Code](https://tommcfarlin.com/php-codesniffer-in-visual-studio-code/)
- [vscode-phpcs](https://marketplace.visualstudio.com/items?itemName=ikappas.phpcs) : extension Phpcodesniffer pour vscode
- [vscode-phpcbf](https://marketplace.visualstudio.com/items?itemName=persoderlind.vscode-phpcbf) : extension pour configurer `phpcbf` script frère de `phpcs` pour beautifer et fixer le code PHP selon le standard choisi