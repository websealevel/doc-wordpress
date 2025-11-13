# doc-wordpress

Un survol de la documentation WordPress et de quelques unes de ses API, de bonnes pratiques à adopter, conseils et références.

- [doc-wordpress](#doc-wordpress)
	- [Abréviations](#abréviations)
	- [Must used plugins](#must-used-plugins)
	- [CPT(Custom Post Types) et CT (Custom Taxonomies)](#cptcustom-post-types-et-ct-custom-taxonomies)
	- [Template hierarchy](#template-hierarchy)
		- [index.php](#indexphp)
		- [singular.php](#singularphp)
		- [single.php](#singlephp)
		- [single-post.php](#single-postphp)
		- [single-{posttype}.php](#single-posttypephp)
		- [home.php](#homephp)
		- [front-page.php](#front-pagephp)
		- [archive.php](#archivephp)
		- [category.php](#categoryphp)
		- [attachment.php](#attachmentphp)
		- [${mimetype}.php](#mimetypephp)
		- [page.php](#pagephp)
		- [${custom}.php](#customphp)
		- [404.php](#404php)
		- [search.php](#searchphp)
		- [archive-${posttype}.php](#archive-posttypephp)
		- [single-${posttype}.php](#single-posttypephp-1)
		- [taxonomy.php](#taxonomyphp)
		- [taxonomy-${taxonomy}.php](#taxonomy-taxonomyphp)
		- [Commentaires](#commentaires)
	- [Taxonomies](#taxonomies)
		- [Introduction](#introduction)
		- [Taxonomy hierarchy](#taxonomy-hierarchy)
		- [Taxonomy custom](#taxonomy-custom)
		- [Taxonomy permaliens](#taxonomy-permaliens)
		- [Divers](#divers)
	- [Permalinks (SEO)](#permalinks-seo)
		- [Slug](#slug)
		- [URL best practices](#url-best-practices)
		- [Outils SEO Wordpress](#outils-seo-wordpress)
	- [Pagination](#pagination)
	- [Template-parts](#template-parts)
	- [Post formats](#post-formats)
		- [Usage](#usage)
	- [Validation, sanitization et échappement](#validation-sanitization-et-échappement)
		- [Validation : Méthodologie proposée, exemple en PHP](#validation--méthodologie-proposée-exemple-en-php)
		- [Sanitization dans Wordpress](#sanitization-dans-wordpress)
	- [Échappemnt (output)](#échappemnt-output)
		- [Pour envoyer des données vers le client (navigateur)](#pour-envoyer-des-données-vers-le-client-navigateur)
		- [Pour envoyer des données vers une base de données](#pour-envoyer-des-données-vers-une-base-de-données)
		- [Escaping dans Wordpress](#escaping-dans-wordpress)
		- [Avec localization](#avec-localization)
	- [Localization](#localization)
	- [Javascript dependencies](#javascript-dependencies)
		- [Jquery](#jquery)
	- [Template tags](#template-tags)
		- [Include tags](#include-tags)
		- [Login tags](#login-tags)
		- [Bloginfo tags](#bloginfo-tags)
		- [Archive tags](#archive-tags)
		- [Calendar tags](#calendar-tags)
		- [Misc tags](#misc-tags)
		- [Navigation tags](#navigation-tags)
		- [Link tags](#link-tags)
	- [Search](#search)
	- [Sécurité](#sécurité)
		- [En bref](#en-bref)
	- [Configuration](#configuration)
		- [Quelques configures du `wp-config.php`](#quelques-configures-du-wp-configphp)
		- [.htaccess](#htaccess)
	- [Debugging](#debugging)
	- [Maintenance](#maintenance)
	- [Settings API : build your own administrative interfaces the core way](#settings-api--build-your-own-administrative-interfaces-the-core-way)
		- [Creer une page d'options](#creer-une-page-doptions)
		- [Définir des options](#définir-des-options)
		- [Définir des sections](#définir-des-sections)
		- [Enregistrer les settings sur une section](#enregistrer-les-settings-sur-une-section)
		- [Ecrire le markup de notre champ (et subtilités)](#ecrire-le-markup-de-notre-champ-et-subtilités)
		- [Construire la page](#construire-la-page)
	- [Quelques plugins recommandés](#quelques-plugins-recommandés)
	- [Optimiser Wordpress (scalability et performances)](#optimiser-wordpress-scalability-et-performances)
	- [Ressources](#ressources)
		- [Doc officielle wordpress.org](#doc-officielle-wordpressorg)
		- [Articles](#articles)
			- [Taxonomies](#taxonomies-1)
		- [Livres](#livres)
		- [Podcasts](#podcasts)
		- [Starter themes](#starter-themes)
		- [Développement de plugin](#développement-de-plugin)
		- [Settings API : Frameworks/Plugins](#settings-api--frameworksplugins)
		- [Custom Fields](#custom-fields)
		- [Formations](#formations)
			- [Gratuit](#gratuit)
			- [Payant](#payant)
				- [les bases](#les-bases)
				- [Les bases à avancé (hyper complet)](#les-bases-à-avancé-hyper-complet)
## Abréviations

- CPT: Custom Post Type
- CT : Custom Taxonomy

## Must used plugins

Dans le dossier `wp-content/mu-plugins`

- toujours actifs
- utilisateurs ne peuvent pas les désactiver par accident
- chargés avant les plugins normaux

Problèmes

- ils n'apparaissent pas dans les notifs de mise a jour. C'est à nous de penser à les update
- les hooks d'activation et de desactivation ne sont pas executés pour les must used plugins

En gros, *à utiliser avec parcimonie*. Il vaut mieux y mettre des plugins qui bougent pas trop, autonomes et isolés.

## CPT(Custom Post Types) et CT (Custom Taxonomies)

Les CPT et CT **sont de préférence déclarés dans un plugin**. Je préfère les mettre dans **`mu-plugins`** pour être sûr qu'ils ne soient pas désactivés par erreur par l'admin.

## Template hierarchy

**Important !**

La [hierarchie](https://developer.wordpress.org/themes/basics/template-hierarchy/) des templates wordpress, et leur fallbacks (quel template est appelé si celui demandé n'est pas trouvé).

- [Visualisation](https://developer.wordpress.org/files/2014/10/Screenshot-2019-01-23-00.20.04.png)
- [Visualisation interactive](https://wphierarchy.com/)

Pour le lire, a gauche c'est *l'input*, par la qu'arrive la requete en fonction de l'url (modèle MVC). C'est la couleur gris foncé sur le diagramme. Et on descend vers la droite jusqu'à trouver le bon template. Jusqu'a `index.php` si on en trouve aucun.

J'appelle `front` tout template qui se trouve en première ligne de la requête. Ces templates ne sont les fallbacks d'aucun autre template, ils sont en première ligne.

### index.php

- sert de fallback à : tout
- fallback : aucun

### singular.php

Pas toujours nécessaire, peut-être redondant avec index.php.

Pour un single post, page, single attachment. *Utile si un post ou une page ont le même template* (car sert de fallback aux deux)

Les archives n'utilisent pas singular.php

- sert de fallback à : page, single, attachment
- fallback : [index.php](#index.php)

### single.php

Template important. pour afficher **un seul `post` ou CPT**.

- sert de fallback à : `attachment`, `single-post`, `single-${posttype}`
- fallback : [singular.php](#singular.php)

### single-post.php

Même que single, mais seulement pour le post par défaut de wordpress (pas de CPT)
Template pour afficher **un seul `post`**.

- sert de fallback à : 
- fallback : [single.php](#single.php)

### single-{posttype}.php

Même que single, mais seulement pour le CPT posttype. Template pour afficher *un seul `post`*.

- sert de fallback à : `single-${posttype}-${slug}`
- fallback : [single](#single.php)

### home.php

- si on choisit pour la front page 'Your latest posts' on utilise `home.php`
- si on choisit une page statique, on peut créer une page avec le slug 'blog' listant tous les posts (`mon-site/blog`), et l'affecter à la page 'Posts page'.

En gros c'est pas compliqué : par défaut Wordpress nous met en home la liste des posts (`/`). Dans ce cas c'est `home.php` qui est appelée. Si on choisit de mettre une page static à la place, il faut aussi choisir une page pour la liste des posts. Et c'est toujours le template `home.php` qui s'en charge.

- sert de fallback à : front-page 
- fallback : [index](#index.php)

### front-page.php

Voir [ici](https://developer.wordpress.org/themes/basics/template-hierarchy/#front-page-display) l'explication détaillée

- sert de fallback à : aucun (front)
- fallback : [custom](#custom) si existe, sinon `page-${slug}` jusqu'à `page`.


### archive.php

Une archive est une collection d'un type de données.
Catégorie archive, tag archive, author archive, CPT archive etc. Assez générique.

Trop générique.

- sert de fallback à : 
- fallback : [index](#index.php)

### category.php

Une archive pour la catégorie Catégorie (catégorie built in)

- sert de fallback à : aucun (front)
- fallback : [archive](#archive.php)

### attachment.php

*Attachment* se traduit par *fichier joint*.

Tout ce qui est attaché à un post depuis le dossier `medias` (image, video, PDF etc..). C'est un type de post fondamentalement.

- sert de fallback à : mimetype
- fallback : [single](#single.php)


### ${mimetype}.php

Template pour un type de donnée spécifique (image, vidéo, PDF). Par exemple

- text.php pour le texte
- video.php pour un attachment video
- image.php pour un attachment image

- sert de fallback à : ${subtype}
- fallback : [attachment](#attachment.php)


### page.php

Controle les pages statiques (les posts de type page)

- sert de fallback à : page-${slug}
- fallback : [singular](#singular.php)

### ${custom}.php

Peut être utile pour tout *post* (post, page, CPT). Ne sera pas pris en compte par la hierarchie par défaut.

Il faut rajouter dans le header de la page (tout en haut)

~~~php
<?php
// Template Name: Ma page
?>
~~~

Wordpress va scanner ce header, et va le rendre accessible via l'admin. 

Pour rendre un template custom utilisable pour un post, une page et un CPT on peut aussi mettre

~~~php
<?php
// Template Name: Ma page
// Template Post Type: post, page, custom-post-type
?>
~~~

Il faut ensuite dire à la page ou au post explicitement d'utiliser ce template au lieu du template par défaut envoyé par la hierarchie.

- sert de fallback à : aucun (front)
- fallback : [singular](#singular.php)

### 404.php

Rendre ludique le fait d'avoir échoué à trouver une ressource. Et redemander à l'user ce qu'il cherchait.

On peut afficher un formulaire de recherche

~~~php
echo get_search_form();
~~~

### search.php

La recherche sur Wordpress par défaut a pour forme `{domain}/?s=motRecherche`. Template utilisé si le paramètre s est présent dans la query string.

- sert de fallback à : aucun (front). 
- fallback : [index](#index.php)

### archive-${posttype}.php

Archive accessible à l'url `/${posttype}`.

**Il faut s'assurer à la registration que le CPT a bien `has_archive` à `true`**, sinon la hierarchie retombera sur `archive`.

Même si on réécrit le slug du CPT lorsqu'on l'enregistre avec `register_post_type`

~~~php
	$args = array(

		'rewrite'               => array(
			'slug'       => 'projets',
			'with_front' => true,
		),


	);

	register_post_type( 'projet', $args );
~~~

l'archive pour le CPT projet **restera `archive-projet.php`**. Le `rewrite` ne réécrit que le `slug` dans les permaliens, **la hierarchie des templates n'est pas concernée**. Ainsi, pour appeler le template `archive-projet.php` je pourrais demander l'url `{mon-domain}/projets`.

~~~php
function mondomaine_register_cpt_monposttype() {
	/**
	 * Post Type: mon-posttype.
	 */
	$labels = array(
        ...
	);

	$args = array(
        ...
		'has_archive'           => true,
		...
	);

	register_post_type( 'mon-posttype', $args );
}

add_action( 'init', 'mondomaine_register_cpt_monposttype' );



~~~

- sert de fallback à : aucun (front). 
- fallback : [archive](#archive.php)


### single-${posttype}.php

Template pour servir un simple CPT.

- sert de fallback à : `single-${posttype}-${slug}` pour personalise la vue à un CPT en particulier, puis `custom` s'il match 
- fallback : [single](#single.php)

### taxonomy.php

Les template de taxonomy vont être par essence des templates de listing (archives).

Template pour liste de tous les items de toutes les CT

- sert de fallback à : `taxonomy-${taxonomy}-${term}` (template spécifique à un terme) puis `taxonomy-${taxonomy}`
- fallback : [archive](#archive.php)

### taxonomy-${taxonomy}.php

Template pour liste de tous les items d'une CT.

- sert de fallback à : `taxonomy-${taxonomy}-${term}`
- fallback : [taxonomy](#taxonomy.php)

### Commentaires

- ne jamais utiliser `*-${id}` comme template ! Sauf en cas de force majeure. Trop dépendant de la bdd et peu explicite.

## Taxonomies

### Introduction

Les taxonomies sont des catégories qui permettent de regrouper les posts et de créer des liens entre eux.

Par défaut, un post standard aura accès à deux taxonomies (crée à l'installation par WordPress) :

- `Categories` (hierarchical cad que les *terms peuvent avoir aussi des enfants*);
- `Tags`(non hierarchical cad que *les terms ne peuvent pas avoir d'enfants*).

Ces deux taxonomies sont incluses dans Wordpress par défaut pour les *posts*. Mais on peut les supprimer si on veut. Il y'en a quatre en tout, si on compte les `Link Categories` (catégorie pour les liens) et les `Post_Format Categories` (pour les types de post en fonction de leur *mimetype*).

Les termes (terms) sont le éléments d'une taxonomie. Ils peuvent avoir un terme parent (*hierarchical*) ou non.

### Taxonomy hierarchy

A lire du haut vers le bas pour voir le *flow* du templating. Valable pour les taxonomies par défaut (voir la hierarchie général des templates) :

~~~
- taxonomy-{taxonomy}-{term}.php
- taxonomy-{taxonomy}.php
- tag-{slug}.php
- tag-{id}.php
- category-{slug}.php
- category-{ID}.php
~~~

Si on l'applique à la taxonomy `Categories` (slug `category`) :

~~~
- category-{slug}.php
- category-{id}.php
- category.php
- archive.php
- index.php
~~~

Si on l'applique à la taxonomy `tags` (slug `tag`) :

~~~
- tag-{slug}.php
- tag-{id}.php
- tag.php
- archive.php
- index.php
~~~

### Taxonomy custom

On peut créer nos propres taxonomies avec `register_taxonomy( '${slug taxonomy}', array( 'posttype' ), $args )`.

Si on applique la hierarchie à la taxonomy custom :

~~~
- taxonomy-{taxonomy}-{term}.php
- taxonomy-{taxonomy}.php
- taxonomy.php
- archive.php
- index.php
~~~

Prendre des précautions [lorsqu'on choisit un nom pour la taxonomy](https://developer.wordpress.org/reference/functions/register_taxonomy/#more-information), qu'elle ne rentre pas en conflit avec des `[noms reservés par Wordpress](https://core.trac.wordpress.org/browser/trunk/src/wp-includes/class-wp.php). Par exemple ne pas utiliser le mot *action*.


### Taxonomy permaliens

On a par défaut les taxonomies `Catégories` (slug `category`) et `Etiquettes` (slug `tag`) pour les articles (slug `post`). Par défaut on a un *term* "Uncategorized" pour la *taxonomie* `category` et un term "Tags" pour la taxonomie `tag`.

Donc par défaut pour accéder à la liste des posts `uncategorized` l'url sera :

~~~
{`mon-domaine}/category/uncategoried`
~~~

De même, pour accéder à la liste des posts étiquetés `Tags` l'url sera :

~~~
{`mon-domaine}/tag/tags`
~~~

On peut changer le slug `category` et `tag` par défaut par un autre dans `Settings/Permalinks/Optional`.

Pour les taxonomies custom. Imaginons que l'on ait créé un CPT `projet` et une CT `savoir-faire` avec un terme 'ebenisterie'. Comment lister tous les projets d'ébenisterie ?

~~~
{`mon-domaine}/savoir-faire/ebenisterie`
~~~


### Divers

- `hierarchical` : 
    - `true`, traite la taxo comme une catégorie (liste d'items prédéfinis),
    - `false` traite la taxo comme un tag. On rentre des termes séparés par des virgules sur chaque post à la volée. 

Si l'on ne souhaite pas avoir une taxonomie hiérarchique et que l'on désire quand même avoir les *checkboxs* (c'est quand même plus sympa) on peut utiliser le paramètre `meta_box_cb` :

~~~php
$args = array(
		'hierarchical'      => false,
		// utilise les metabox de post comme pour la taxonomy hierarchique 'category'
		'meta_box_cb'       => 'post_categories_meta_box',
	);
register_taxonomy( 'action', array( 'ma_taxonomie' ), $args );
~~~


## Permalinks (SEO)

Le *permalink* dans Wordpress c'est l'**URL** entière d'une ressource (incluant le protocole et le nom de domaine).

Le *SEO* ou Ranking de vos contenus parmi les résulats retournés par votre navigateur est hyper important si vous voulez être trouvé. Qui va sur la 2eme page de Google résultats ? (Montrer stat)

Le SEO c'est souvent de la magie noire (ça depend par exemple de code propriétaire du moteur de Google et les règles peuvent changer très vite) mais c'est *important*. Et *il y a quand même des règles* stables dans le temps.

Essayer de prévoir le SEO `avant` de commencer le développement ou la mise en prod de votre site. Cela vous aidera a la structuration de vos pages. Le faire après peut devenir vite compliqué voire impossible.

Un site bien structuré avec taxonomie et types de posts, des url bien formées et bien nommées, des métadonnées pour les pages et du **contenu de qualité** est la base du SEO.

### Slug

Les slug sont la dernière partie de l'URL (*path*), par exemple dans `example.com/about-us` le slug est `about-us`. Dans `example.com/?p=356` il n'y a pas de slug. L'URL est la ressource racine (/) modulée par un query parameter (`p`).

**Toujours utiliser** `Post name`: plus compréhensible, SEO friendly

- Les moteurs de recherche utilisent votre URL pour déterminer de quoi traite votre page ou votre contenu. Et s'ils trouvent que votre permalien est en lien avec le contenu ou la page, les moteurs de recherche le considèrent comme authentique et légitime, ce qui leur donne la priorité dans les résultats de recherche.

- L'utilisation de mots-clés SEO dans votre URL peut vous aider à vous positionner sur vos mots-clés cibles. Google utilise l'URL comme un facteur pour classer votre page, donc si le slug de l'URL inclut vos mots-clés, Google sera plus enclin à la référencer.

### URL best practices

- Utiliser le protocole HTTPS ;
- Nom de domaine clair ;
- *Keep it short and precise* (url concises). Le poids des derniers mots pour les *bots crawlers* est plus faible
- Lisible pour les robots **et** pour les humains
- Utiliser des sous-domaines : `sous-domaine.mon-domaine.com`
- Placer des mots clefs qui décrivent le contenu de la page
- Éviter de mettre une année dans l'URL
- Éviter la duplication d'URL sur votre site
- Le meilleur moment pour modifier un *permalien* est **lorsque vous créez initialement le contenu du post**, **avant que le post ne soit publié**. Pourquoi ? Si vous changez le *slug* d’un post après sa publication, attention : vous modifiez également l’URL du post, ce qui peut entraîner une page 404. **Tous les liens partagés en ligne utilisant l’ancienne URL ne fonctionneront plus et devront être redirigés vers la nouvelle URL**. En personnalisant le slug de votre post avant de publier le contenu, vous aurez un permalien optimisé dès que vous appuierez sur le bouton *Publier*;
- Structure d'URL cohérente et uniforme sur tout le site 


### Outils SEO Wordpress

 - [Yoast](https://yoast.com/), un des plugins les plus installés et maintenu de l'écosystème de Wordpress. Gratuit possible, *user friendly* pour vos administrateur·ices de contenus (clients). Ne pas hésiter à l'installer.
 - [SEOKey](https://www.seo-key.fr/), plugin français de SEO réputé pour sa qualité.


## Pagination

- [`paginate_links()`](https://developer.wordpress.org/reference/functions/paginate_links/) : on a la main sur le nombre de pages de résultats et le formattage du markup de balisation
- le nombre de posts par page est reglé depuis l'admin dans Settings/Reading
- si on veut faire des choses plus contrôlés on passe par une [WP_Query](https://developer.wordpress.org/reference/classes/wp_query/)


## Template-parts

On crée en général un dossier `template-parts` à la racine du thème. On y stocke tout template html de contenu. Par exemple

- `template-parts/content.php` pour le contenu d'un post
- `template-parts/content-page.php` pour le contenu d'une page
- etc...

Par exemple, le contenu de `content.php` :

~~~php
<?php 
/**
 * Contenu d'un post
 */
?>
<article id="post-<?php the_ID(); ?>" <?php post_class(); ?>>
    <header class="entry-header">
        <?php the_title('<h1>', '</h1>'); ?>
    </header>
    <div class="entry-content">
        <?php the_content(); ?>
    </div>
</article>
~~~

On l'appelle ensuite avec la fonction `get_template_part({path du fichier content},{ce qui suit le -}` dans la *main loop*. Par exemple :

~~~php
<?php if (have_posts()) : while (have_posts()) : the_post(); ?>
            <?php get_template_part('template-parts/content','page') ?>
        <?php endwhile; else : ?>
            <?php get_template_part('template-parts/content', 'none') ?>
<?php endif; ?>
~~~

## [Post formats](https://wordpress.org/support/article/post-formats/)

Pour présenter différents types de posts de manière différente. Les formats sont des sortes de métadonnées sur les posts. Par défaut un post a le format `standard`. Utile pour utiliser des *template-parts* en fonction d'une méta (ex: vidéo, comment etc...)

Les noms des formats match les noms des dashicons.

### Usage

~~~php
if ( has_post_format( 'video' )) {
  echo 'this is the video format';
}
~~~

~~~php
echo get_post_format($post_id);
~~~

Les post-formats ont un [template dans la hierarchie](https://developer.wordpress.org/themes/basics/template-hierarchy/#custom-taxonomies) du type `taxonomy-post_format-post-format-{format}.php`.

Mais à utiliser *davantage pour le style que pour du templating*. On peut par exemple définir un `content-{format}.php` dans nos `template-parts` et charger un template différent en fonction du format.

À utiliser pour le format `quote` pour les `testimonials` par exemple, au lieu d'un CPT !

## Validation, sanitization et échappement

*Process of cleaning or filtering your input data*

On parle aussi de *sanitazing*, validating, filtering, cleaning... Même si on peut dire qu'il y a des différences entre ces termes globalement ils veulent tous dire "preventing invalid from **entering** your application.

### Validation : Méthodologie proposée, exemple en PHP

Compare la donnée à une liste de données acceptées (inputs *fermés*). **Le cas idéal**, à privilégier dès que possible.

Un ensemble de données `$data` arrive.

- Crée un tableau $clean vide. Ce tableau ne contiendra que des données dont on est sûrs qu'elles ont été validées.
- Creer un ensemble de **regles de validation** que $data doit passer avant de se retrouver dans $clean.
- Privilégier des fonctions builtin quand c'est possible (plus sur que votre code)
- Identifier l'input (donnée considérée comme invalide et sale)
- Valider l'input
- Distinguer entre donnée sale et donné valide

### Sanitization dans Wordpress

Sanitize strips ou modifie les data, c'est a dire qu'il **modifie la donnée**. 

C'est donc du filtre qui peut ouvrir vers des vulnérabilités. Il faut faire confiance à une fonction maintenue par une communauté depuis 20ans.

[Fonctions offertes par WordPress](https://developer.wordpress.org/apis/security/sanitizing/#sanitization-functions) :

~~~php
sanitize_email()
sanitize_file_name()
sanitize_hex_color()
sanitize_hex_color_no_hash()
sanitize_html_class()
sanitize_key()
sanitize_meta()
sanitize_mime_type()
sanitize_option()
sanitize_sql_orderby()
sanitize_term()
sanitize_term_field()
sanitize_text_field()
sanitize_textarea_field()
sanitize_title()
sanitize_title_for_query()
sanitize_title_with_dashes()
sanitize_user()
sanitize_url()
~~~

## Échappemnt (output)

**Très important.**

Échaper ou encoder des caractères pour être sûr que leur signification originale est préservée sans que ces caractères soient interprétés dans le contexte donné (HTML, SQL, etc.)

Processus en 3 étapes:

- Identifier les output
- Échapper les outputs
- Distinguer entre donnée échappée et non échappéé

### Pour envoyer des données vers le client (navigateur)

[`htmlentities`](https://www.php.net/manual/fr/function.htmlentities.php) est la meilleure fonction native PHP pour échapper les données dans le contexte HTML. 

Un exemple :

~~~php

//identification de notre sortie
$html=array();

//échappement
$html['username'] = htmlentities($clean['username'], ENT_QUOTES, 'UTF_8');

//servi au client
echo "<p>Welcome back, {$html['username']}.</p>"
~~~

### Pour envoyer des données vers une base de données

Utiliser l'API de WordPress pour faire des **requêtes préparées**.

### Escaping dans Wordpress

Wordpress met à disposition tout un tas de fonctions pour l'échappement. Vous n'aurez jamais besoin d'utiliser htmlentities directement.

*Output shoudl occured as late as possible*. Après que tous les filtres WP aient été appliquées. En gros, juste avant de l'echo sur la sortie standard.

- **esc_html** : a utiliser à la place de htmlentities
- esc_url: converse caracteres dans encoding url
- esc_js: rend <script></script> inofensif
- esc_attr: clean les attributs d'element html
- esc_textarea: pareil que esc_html mais dans un text area

Avant de renvoyer une donnée dans un script php sur la sortie standard, on veut être sûr de pas envoyer du code malfaisant au client, et éviter du cross-site-scripting (XSS). 

[Voir la liste de fonctions utiles d'échappement](https://developer.wordpress.org/themes/theme-security/data-sanitization-escaping/#escaping-securing-output)


### Avec localization

Voir [ici](https://developer.wordpress.org/themes/theme-security/data-sanitization-escaping/#escaping-with-localization)

- `_e(text, domain)`, **print** une string traduite
- `__(text,domain)`, **renvoie** une string traduite sans la print

Pour échapper ces fonctions de traductions on fait soit par exemple `esc_html(__(text,domain))`. Soit en plus court (donc à privilégier) `esc_html_e(text, domain))`.

~~~php
esc_html_e( 'Hello World', 'text_domain' );
// same as
echo esc_html( __( 'Hello World', 'text_domain' ) );
~~~

## Localization

Pour la traduction.

- **esc_html_e** : escape html, translate puis echo. A utiliser de préférence tout le temps !
- esc_html__ : escape html; translate and return string
- esc_attr_e : escape attribute, translate puis echo
- _e : translate and echo (pas d'escape)
- __ : translate et retourne (pas d'escape)

Toutes ces fonctions prennent deux arguments :

~~~php
_e('text a traduire', 'nom-**unique**-de-domaine');
~~~

`nom **unique** de domaine` vient du `Text Domain` de notre thème, définit obligatoirement dans le `style.css` de notre thème, avec toutes les autres métas du theme.

Voir ce [lien](https://developer.wordpress.org/apis/handbook/internationalization/localization/) pour comprendre comment fonctionne en pratique la localization.

## Javascript dependencies

Si on regarde la documentation de la fonction [wp_enqueue_scripts](https://developer.wordpress.org/reference/functions/wp_enqueue_script/#default-scripts-and-js-libraries-included-and-registered-by-wordpress) on peut voir toutes les dépendances JS du coeur de Wordpress (utilisé côté *admin*). 

Si on a besoin de l'une de ces librairies **on a pas besoin de les importer explicitement dans notre projet**. Il suffit de copier leur slug dans le tableau `$deps` passé en argument de `wp_enqueue_scripts`.

### Jquery

Si on veut par exemple embarquer `jquery` sur nos pages webs servies au client :

~~~php
wp_enqueue_script( 'main-script', get_stylesheet_directory_uri() . '/js/scripts.min.js', array( 'jquery' ), $js_version, true );
~~~

Dans notre fichier `main-script.js` si on veut utiliser jquery, **il ne faut pas utiliser le signe $ car il peut rentrer en conflit avec d'autres library**.

Donc au lieu d'écrire :

~~~javascript
$('.single-projet)
~~~

on écrira :

~~~javascript
jQuery('.single-projet)
~~~

## Template tags

Les [`template tags`] sont des fonctions spéciales de wordpress qui permettent d'accéder au contenu/données wordpress (par ex `get_the_title()`, `the_title()`) . Elles passent aussi par des filtres (hook filter) `add_filter('the_title', callback)`.

La plupart de ces fonctions acceptent des paramètres.

~~~php
the_title('<h1>','</h1>');
~~~

### Include tags

- get_header()
- get_footer()
- get_sidebar()
- get_template_part()
- get_search_form()
- comments_template()

### Login tags

- wp_loginout() : pour fair un bouton de connexion/deconnexion facilement. Si on veut être redirigé vers la même page où est présent le bouton on peut faire `<?php wp_loginout( get_permalink() ?>`. La fonction prend en argument une url pour le `redirect` après login ou logout.
- wp_logout_url()
- wp_login_url()
- wp_login_form()
- wp_lostpassword_url()
- wp_register()
- is_user_logged_in()

### Bloginfo tags

- bloginfo(), voir tous les paramètres

### Archive tags

Pour les listings.

- single_post_title(), outside of the loop
- post_type_archive()
- single_cat_title()
- single_tag_title()
- single_term_title()
- **wp_get_archives()** : très utile pour faire des listings par date, par postype très facilement (pareil que le widget, editable par l'user)

Par exemple, créer une liste de liens de poste projets par année, 3 max en affichant le nombre de posts par année
~~~php
<?php
$args = array(
	'type'            => 'yearly',
	'limit'           => 3,
	'show_post_count' => true,
	'post_type'       => 'projet',
);
wp_get_archives( $args );
?>
~~~

### Calendar tags

- **get_calendar()** (pareil que le widget, editable par l'user)
- calendar_week_mod()
- delete_get_calendar_cache()

### Misc tags

- wp_meta()
- get_current_blog_id() (pour le multisites)
- wp_title (deprecated, auto display a present)
- allowed_tags()
- wp_enqueue_scripts()

### Navigation tags

- [wp_nav_menu](https://developer.wordpress.org/reference/functions/wp_nav_menu/): display a menu on the page. Utiliser tous les args pour les wrapper au lieu de faire du markup HTML a la main autour.
- [register_nav_menu](https://developer.wordpress.org/reference/functions/register_nav_menus/): ajouter des locations de menu
- walk_nav_menu_tree (used internally by the core) utilisé pour controler comment les menus sont construits et affichés

### Link tags

- the_permalink(): a connaitre
- get_permalink(): a connaitre 
- get_post_permalink()
- get_page_link()
- get_attachment_link()
- edit_post_link()
- get_edit_post_link()
- get_delete_post_link()
- edit_comment_link()
- edit_tag_link()
- home_url()
- site_url()
- get_search_url($terme_recherche): Retrieves the permalink for a search. 
- get_search_query()
- the_feed_link()

## Search

2 notations équivalentes dans l'url

~~~
{mon-domain}/?s=terme-recherche
~~~

ou

~~~
{mon-domain}/search/terme-recherche
~~~

## Sécurité

### En bref

- Rester à jour (core, plugins, themes)
- Modifier le prefixe des tables de wordpress, ne pas utiliser `wp_`
- Limiter les tentatives de login
- Faire une whitelist des IP qui peuvent se connecter en tant qu'admin (dans le .htaccess)
- Utiliser des mots de passe forts
- Déplacer le fichier de configuration `wp-config.php` en dehors de l'install de wordpress. On peut le placer en dehors du root de Wordpress. Wordpress va checker automatiquement le dossier parent s'il trouve pas le wp-config dans son dossier d'install. Sur Apache, pour eviter qu'on puisse acceder au `wp-config.php` en plain/text (suite a un bug) on peut rajouter la regle `<FileMatch ^wp-config.php$>deny from all</FilesMatch>` au `.htaccess`
- Deplacer le repertoire wp-content ailleurs que dans l'install de wordpress. Cela ne rend pas le site plus secure mais ça permet d'échapper à bcp de scripts d'attaque car la structure ne sera plus standard. Ajouter dans le `wp-config.php` 
    ~~~php
    define('WP_CONTENT_DIR', $_SERVER['DOCUMENT_ROOT'] . '/mon-site/wp-content';
    define('WP_CONTENT_URL', 'http://example/com/mon-site/wp-content');
    define('WP_PLUGIN_URL', 'http://example.com/mon-site/wp-content/plugins');
    ~~~
- Utiliser les rôles (*least privileges)
- Utiliser les salts
- Forcer le SSL au login et aux admins avec `define('FORCE_SSL_LOGIN', true);` et `define('FORCE_SSL_ADMIN', true);`
- Utiliser un plugin spécialisé en sécurtié comme [BulletProof Security (BPS)](https://forum.ait-pro.com/read-me-first/) ou WordFence.
  
## Configuration

### Quelques configures du `wp-config.php`

~~~php
define('WP_ENVIRONMENT_TYPE', 'production');
define('WP_DEBUG', true);          // Activé pour que les erreurs soient loguées
define('WP_DEBUG_LOG', true);      // Consigne les erreurs dans wp-content/debug.log
define('WP_DEBUG_DISPLAY', false); // RIEN AFFICHER SUR LA SORTIE (ECRAN)
define('SAVEQUERIES', false);      // Surtout pas !
define('SCRIPT_DEBUG', false);     // Désactive le chargement de assets du core non minifiées
// Active/Desactive toutes les maj auto (core, themes, plugins, translations).
define( 'AUTOMATIC_UPDATED_DISABLED', false );
// Maj auto pour le core de wp seulement: false, true ou minor.
define( 'WP_AUTO_UPDATE_CORE', 'minor' );
// Limite le nombre de revisions max pour un post.
define( 'WP_POST_REVISIONS', 3 );
// Limite l'intervalle entre 2 autosave de l'éditeur en secondes. Par défaut 60s.
define( 'AUTOSAVE_INTERNAL', 120 );
// Memoire RAM max alloué à Wordpress par le serveur (si le host est d'accord)
define('WP_MEMORY_LIMIT', '64MB');
// Active le cache, utile pour certains plugins pour optimiser le site
define('WP_CACHE', true);
//Forcer le SSL pour le login
define('FORCE_SSL_LOGIN', true);
//Forcer le SSL pour servir toutes les pages d'admin (/wp-admin). Ralentit le chargement des pages d'admin mais assure le chiffrement de toutes les données.
define('FORCE_SSL_ADMIN', true);
~~~

### .htaccess

Le fichier `.htaccess` est un fichier de configuration d'Apache. On peut y définir des règles et c'est là que réside la magie de wordpress

~~~.htaccess
RewriteCond ${REQUEST_FILENAME} !-f
RewriteCond ${REQUEST_FILENAME} !-d
RewriteRule . /index.php [ L]
~~~

Ce `.htaccess` dit *si la requete ne demande ni un fichier, ni un repertoire qui existe sur le path alors renvoie la requete vers le index.php*, ce dernier lancant tout le core de wordpress.

On peut se servir du .htaccess pour rajouter de la config

- redirections permanentes: `301 redirect 301 {ancienne-url} {nouvelle-url}`
- mémoire allouée à PHP : `php_value memory_limit 64M`
- taille des uploads max: `php_value upload_max_filesize 20M` et `php_value post_max_size 20M`
- sécurtié, restreindre l'accès à des IP.

Bonne pratique : restreindre l'accès au dossier `wp-admin` a une liste blanche d'ip. Pour cela, creer un `.htaccess` dans le dossier `wp-admin` avec le code suivant

~~~.htaccess
AuthType Basic
order deny,allow
deny from all
#IP adress to whitelist
allow from xxx.xxx.xxx.xxx.
~~~~

Ainsi, seule une liste blanche d'IP d'admin peuvent se connecter en tant qu'admin au Wordpress. Si l'admin a une nouvelle IP il suffit de l'ajouter à la liste.

Configuration de log d'erreur pour un site en production directement dans le `.htaccess` :

~~~.htaccess
php_flag display_startup_errors off
php_flag display_errors off
php_flag html_errors off
php_flag log_errors on
php_value error_log /public_html/php_errors.log
~~~

où `error_log` est une valeur relative au *Document root* du serveur web Apache (du Virtual Host) et non la racine de l'installation de Wordpress.

## Debugging

~~~php
// Active le mode debug - equivalent à  php_flag log_errors on
define( 'WP_DEBUG', true );
// Ne pas afficher les messages d'erreur sur la sortie standard.
define( 'WP_DEBUG_DISPLAY', true );
// Log les erreurs dans le fichier debug.log.
define( 'WP_DEBUG_LOG', true );
// Log toutes les requetes vers la db dans un tableau,
define( 'SAVEQUERIES', true );
// Pour afficher le tableau de la requête
global $wpdb;
print_r( $wpdb->queries );
//Afficher toutes les constantes PHP disponibles
print_r( get_defined_constants() );
~~~

## Maintenance

Il y a un mode maintenance natif à Wordpress. Il suffit de créer un fichier `.maintenance` à la racine de l'installation de wordpress et y mettre le code suivant

~~~php
<?php
 $upgrading = time();
?>
~~~

On peut customiser davantage cette page en créant un fichier `maintenance.php` dans le `wp-content`. Celui-ci prend la priorité sur le fichier `.maintenance`

## Settings API : build your own administrative interfaces the core way

Utiliser pour développer des plugins et des pages d'administration/d'options custom contenant des formulaires. On peut faire vraiment tout ce qu'on veut avec et le fait d'utiliser cette méthode plutot que des hacks (notamment via des pages d'options ACF (PRO ONLY!)) : donnent plusieurs avantages

- visual consistency
- robustness and future proof
- less work : handling form submission, security nonce, sanitizing data for free (une fois qu'on a compris comment ça marchait)

Cette API est très bien mais elle est vraiment pas intuitive et demande pas mal de code boilerplate pour être effective. Cela dit elle est native, supportée et encouragée pour développer des plugins, des pages d'options d'un thème de manière professionnelle.

Il y'a d'autres alternatives, comme des frameworks qui encapsulent tout ça mais comme tout usage de code tiers il faut faire attention à ce que ces frameworks soient bien maintenus dans le temps et soient capables d'accomoder les éventuelles évolutions de la Settings API. Voir des ressources [ici](#settings-api-frameworkstools).

Dans tous les cas, avant d'utiliser des outils pour vous épargner beaucoup de travail redondant il est bien de le faire au moins une fois pour savoir qu'est ce qui vous ne plait pas pour trouver éventuellement le bon outil pour vous facilier la vie, ou créer le votre.

### Creer une page d'options

~~~php
//creation d'un menu custom
add_action('admin_menu','domain_create_menu');

function domain_create_menu(){

//creation d'un menu dans le menu de premier niveau 'Settings'
add_options_page('Mon plugin','Mon plugin','manage_options','domain-options','domain_settings_page');

//appeler la fonction pour enregistrer les settings
add_action('admin_init','domain_settings_page_html');

}
~~~

### Définir des options


~~~php
function domain_settings_page(){
register_setting(
    'domain-settings-group',
    'domain_options',
    array(
            //La valeur par défaut, tres important au premier lancement quand l'option n'existe
            //pas encore en base, dit ce que get_option() doit retourner par défaut
            'default' => array(
				'domain_setting_tel' => '02 02 02 02 02',
			),
            // La fonction appelée après la soumission du formulaire, ici on clean et on valide
			'sanitize_callback' => function ( array $input ) {
                //Sanitize ici chaque champ
                //$input['domain_setting_tel'] = filtrer($input)
				return $input;
			},
    )
);

}
~~~

`register_settings` dit à l'Options API qu'elle va enregistrer quelque chose sous la clef `domain_options`. Ces options seront enregistrées dans la table `wp_options` sous la forme de paires clé valeur.

On va avoir plusieurs champs, plusieurs options à définir mais on va les enregistrer sous la même clef dans la table sous forme de tableau serialisé. Le premier paramètre est l'`options group name`, il permet d'identifier toutes les options. Le deuxième paramètre est important car c'est la clef sous laquelle sera enregistrée les options dans la table. Il doit être unique. Le troisième paramètre est la callback pour sanitizer les valeurs de nos options.

### Définir des sections

~~~php
function domain_settings_page(){

	add_settings_section( 
    'domain-section-section1', 
    'Site', 
    'domain_setting_section1_html', 
    'domain-options' 
);

}
~~~

On enregistre une section sur la page `domain-options` (id de la page). La callback `domain_setting_section1_html` sert à afficher du markup supplémentaire pour notre section (en général de l'aide).

### Enregistrer les settings sur une section

Pour boucler le tout on doit dire à la section quels settings (champs) elle va présenter. On le fait avec [`add_settings_field`](https://developer.wordpress.org/reference/functions/add_settings_field/). On enregistre ici un champ de numéro de téléphone que l'on appelera `domain_setting_tel`. Il sera enregistré comme une clef de l'option `domain_options`, enregistrée en base, qui est, je vous le rappelle, un tableau sérialisé.

~~~php
add_settings_field(
		'mon_champ_id',
		'Numéro de telephone',
		'domain_settings_tel_callback',
		'domain-options',
		'domain-section',
		array(
			'label_for' => '',
			'class'     => '',
		)
	);
~~~

Ok, je vous laisse consulter la doc pour savoir ce que tout ces arguments signifient. En gros, on dit j'enregistre un champ avec
- `mon_champ_id` comme id du champ (pas vraiment important)
- `Numéro de telephone` qui sera affiché comme label pour la champ
- `domain_settings_tel_callback`, la callback pour ecrire notre input
- `domain-options`, l'id ou slug de la page sur laquelle on enregistre le champ
- `domain-section`, l'id ou slug de la section sur laquelle on enregistre le champ
- `array()`, le dernier argument est en option, pour passer des trucs en plus à notre callback

Vous remarquerez qu'on a pas utilisé ici le nom du champ `domain_setting_tel`. C'est parce que c'est une clef de l'option `domain_options`. Donc c'est à nous de joueur un peu avec la Settings API. Je sais c'est pas l'API la plus simple de Wordpress...

### Ecrire le markup de notre champ (et subtilités)

Bientôt fini, écrivons le markup de notre champ en définissant notre fonction `domain_settings_tel_callback`.

~~~php
/**
 * Construit l'input domain_setting_tel
 *
 * @param array $input Données injectées via add_settings_field.
 */
function adb_setting_nb_max_featured_projects_callback( $input ) {

    //On récuperer la valeur en base. Si elle existe pas encore, on récupere la valeur renseignée sous la clef 'default' dans `register_setting`
	$values = get_option( 'domain_options' );
	$value  = $values['domain_setting_tel'];
	?>
	<input 
	id="domain_setting_tel_id"
	type="text" 
	name="<?php echo esc_attr( 'domain_options[domain_setting_tel]' ); ?>" 
	value="<?php echo esc_attr( $value ); ?>"
	>
	<?php
}
~~~

Voilà ! Quand on postera le formulaire, tout devrait bien se passer. La callback de sanitization devrait être appelée et ensuite l'option mise à jour en base. C'est la Settings API qui se charge de tout ça. Nous on doit juste bien cabler le tout.

Il ne reste qu'à construire la page d'options à proprement dit.

### Construire la page

On définit notre fonction `domain_settings_page_html` qui va construire notre page d'options (markup)

```php
function domain_settings_page_html(){
?>
<div class="wrap">
	<h2>Mon plugin</h2>
	<form action="options.php" method="post">
        // securité : creation auto de nonce par wordpress pour le groupe d'options
		<?php settings_fields( 'domain-settings-group' ); ?>
        // génération des sections présentes sur la page
		<?php do_settings_sections( 'domain-options' ); ?>
        // soumettre le formulaire
		<p class="submit">
			<?php submit_button( 'Enregistrer les changements' ); ?>
		</p>
	</form>
</div>
}
<?php
}
```
De la magie a lieu ici sur plusieurs lignes

- `do_settings_sections( 'domain-options')` : la Settings API rend tout le markup des sections et des champs qui y sont enregistrés ;
- `settings_fields('domain-settings-group')` : va intégrer des inputs cachés, des nonces, pour valider l'origine de la soumission du formulaire et nous protéger d'attaque CSRF. Rien à faire de plus de notre part.

Quand on soumet un formulaire de settings, la Settings API se charge de :

- Valider le nonce ;
- Valider/sanitizer chaque champ avec la déclaration de notre `sanitize_callback` ;
- Mettre à jour l'option en base et sauvegarder les inputs.

Voilà, un *aperçu* de la Settings API.

Comme vous pouvez le voir c'est assez puissant mais* c'est hyper verbeux et c'est facile de se perdre car elle est pas si intuitive que ça*. 

Je recommande donc après l'avoir fait une fois de manière *vanilla* de s'écrire un petit plugin ou une petite fonction *utils* pour aider à préparer tout ça de manière beaucoup plus simple et efficace.


## Quelques plugins recommandés

- [W3 Total Cache](https://www.boldgrid.com/w3-total-cache/) : permet de mettre en cache les pages générés par Wordpress. Crée un dossier wp-content/cache et y stocke du html static généré par les requetes précédentes. La prochaine fois qu'une page est appelée, au lieu d'executer la loop et de faire des requetes a la base, wordpress sert la page static déjà générée. Ameliore le SEO, Lazy Image etc...
- [WordFence](https://www.wordfence.com/)
- [ACF](https://www.advancedcustomfields.com/), la base pour manipuler les post metas. Ne pas hésiter à passer sur la version PRO au besoin, elle vaut le coup.

## Optimiser Wordpress (scalability et performances)

- Hebergeur adapté
- Mettre en place une mise en cache
- Ne pas uploader vidéos/audio sur le site. Mettre sur youtube, soundcloud et utiliser le lien plutot. Ca augmente la taille des backups, ca coute cher en bande passante
- Reduire les appels à la db (mieux vaut remonter plus d'infos en une fois que une info puis une info puis une info)
- Limite le nb de revisions de posts. Dans le `wp-config.php` : `define( 'WP_POST_REVISIONS', 3 );`
- Disable hotlinking and leaching of your content
~~~.htaccess
#disable hotlinking of images with forbidden or custom image option
RewriteEngine on
RewriteCond %{HTTP_REFERER} !^$
RewriteCond %{HTTP_REFERER} !^http(s)?://(www\.)?{domaine} [NC]
RewriteCond %{HTTP_REFERER} !^http(s)?://(www\.)?google.com [NC]
RewriteRule \.(jpg|jpeg|png|gif)$ – [NC,F,L] 
~~~
- Limiter la taille des postes/pages
- Afficher l'excerpt plutot que tout le post dans les listings
- Reduire le nombre de plugins
- Au lieu de link vers des CDN (font, icons, js lib) les mettre directement sur le site
- Optimisez les images pour le web (compression, taille)
- Utiliser le lazy loading
- Configurer PHP via le `php.ini`, desactiver les extensions inutiles pour wordpress
~~~php.ini
;security
expose_php = Off
;performence
register_argc_argv = Off
;upload
file_uploads = On
memory_limit = 128M
upload_max_filesize = 10M
post_max_size = 48M
max_execution_time = 600
max_input_vars = 1000
;Opcache
opcache.memory_consumption=64
opcache.interned_string_buffers=16
opcache.max_accelerated_files=600
opcache.validate_timestamps=0
opcache.revalidate_freq=0
opcache.fast_shutdown=1
~~~

## Ressources

### Doc officielle wordpress.org

Très bien faite, mais peut parfois demander un peu d'experience pour s'y retrouver 

- [Developer Resources](https://developer.wordpress.org/), point d'entrée de la documentation officielle
- [Wordpress hierarchy](https://developer.wordpress.org/themes/basics/template-hierarchy/)
- [Wordpress coding standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/)
- [Using Permalinks](https://wordpress.org/support/article/using-permalinks/)
- [Data Sanitization/Escaping](https://developer.wordpress.org/themes/theme-security/data-sanitization-escaping/#escaping-with-localization)
- [Localization](https://developer.wordpress.org/apis/handbook/internationalization/localization/)
- [Codex](https://codex.wordpress.org/), **wiki en cours de dépréciation**
- [Template tags](https://codex.wordpress.org/Template_Tags)

### Articles

#### Taxonomies

- [WordPress Taxonomies: The Ultimate Guide](https://ithemes.com/blog/wordpress-taxonomies)
- [WordPress Permalinks: The Essential Guide](https://ithemes.com/blog/wordpress-permalinks)

### Livres

- *Professional WordPress: Design and Development* de Brad Williams et David Damstra, Edition Wrox, 3rd Edition, 2015
- *Professional WordPress Plugin Development* de Brad Williams et Justin Taldock, Edition Wrox, 2nd Edition, 2020
- *Essential PHP Security*, Chris Shiflett, Edition O'REILLY, 2006 (Attention certaines fonctions PHP sont deprecated mais sinon la philosophie et les conseils sont toujours pertinents et bien expliqués. Lecture 'attentive').
- *Modern PHP: new features and good practices*, Josh Lochart, Edition O'REILLY, 2015

### Podcasts

- [HTML All The Things - Web Development, Web Design, Small Business](https://podcast.htmlallthethings.com/e/the-thing-about-wordpress/) 

### Starter themes

- [Underscores](https://underscores.me/)

### Développement de plugin

- [Plugingplate](https://pluginplate.com/plugin-boilerplate/) : starterpack
- [Introduction](https://developer.wordpress.org/plugins/intro/)

### Settings API : Frameworks/Plugins

A vos risques et périls

- [5 Ways to Create a WordPress Plugin Settings Page ](https://deliciousbrains.com/create-wordpress-plugin-settings-page/)
- [WordPress Settings Framework](https://github.com/iconicwp/WordPress-Settings-Framework)
- [Creating A WordPress Settings Page Using the WordPress REST API](https://torquemag.io/2017/06/creating-wordpress-settings-page-using-wordpress-rest-api/)
- [wp-optionskit](https://github.com/WPUserManager/wp-optionskit)
- [Settings API Code Generator](http://wpsettingsapi.jeroensormani.com/)

### Custom Fields

- [ACF](https://www.advancedcustomfields.com/), classic et solide. Mais payant pour avoir accès a des features hyper utiles comme des champs repeater ou des pages d'options
- [Carbon](https://carbonfields.net/), meilleure alternative à ACF. Complètement gratuit, solide et robuste. Champs repeaters et gallery gratuit !

### Formations

- [Zack Gordon (le daron)](https://zacgordon.com/)

#### Gratuit

- [Cours wordpress.org](https://learn.wordpress.org/courses)
- [Building websites with WordPress](https://nmiletic.gumroad.com/l/kSrqD) 
- [Learn Wordpress](https://kinsta.com/learn/)
- [Wordpress for beginners training](https://yoast.com/academy/free-training-wordpress-for-beginners/)
- [How to Learn WordPress for Free in a Week (or Less)](https://twitter.com/natmiletic/status/1511711827398258695)
- [Wordpress tutorials](https://www.siteground.com/tutorials/wordpress/)
- [How to](https://wordpress.tv/category/how-to/), video
- [Our wordpress library](https://wpapprentice.com/courses/)
- [Wordpress freecodecamp](https://www.freecodecamp.org/news/tag/wordpress/)
- [Building PHP MVC Framework from Scratch](https://www.youtube.com/watch?v=WKy-N0q3WRo&list=PLLQuc_7jk__Uk_QnJMPndbdKECcTEwTA1), The Codeholic. Pas sur Wordpress mais montre les fonctionnalités de base d'un framework à implémenter

#### Payant

##### les bases

- [Become a WordPress Developer: Unlocking Power With Code](https://www.udemy.com/course/become-a-wordpress-developer-php-javascript/), pas mal pour apprendre le processus de la création de thème dans son ensemble

##### Les bases à avancé (hyper complet)

- [Complete WordPress Theme & Plugin Development Course [2021]](https://www.udemy.com/course/wordpress-theme-and-plugin-development-course), thèmes classiques
- [WordPress REST API Complete Beginners Guide](https://www.udemy.com/course/wordpress-rest-api-course/)

