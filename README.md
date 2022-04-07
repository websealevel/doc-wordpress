# doc-wordpress

[TOC]
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
		- [Divers](#divers)
	- [template-parts](#template-parts)
	- [Post formats](#post-formats)
		- [Usage](#usage)
	- [Escaping output](#escaping-output)
		- [Sécurité](#sécurité)
		- [Avec localization](#avec-localization)
	- [Javascript dependencies](#javascript-dependencies)
		- [Jquery](#jquery)
	- [Template tags](#template-tags)
	- [Include tags](#include-tags)
	- [Login tags](#login-tags)
	- [Bloginfo tags](#bloginfo-tags)
	- [Archive tags](#archive-tags)
	- [Calendar tags](#calendar-tags)
	- [Misc tags](#misc-tags)
	- [Ressources](#ressources)
		- [Doc officielle](#doc-officielle)
		- [Articles](#articles)
			- [Taxonomies](#taxonomies-1)
		- [Livres](#livres)
		- [Podcasts](#podcasts)
		- [Développement de thèmes](#développement-de-thèmes)
		- [Développement de plugin](#développement-de-plugin)
		- [Formations](#formations)
			- [Gratuit](#gratuit)
			- [Payant](#payant)
				- [les bases](#les-bases)
				- [les bases à avancé (hyper complet)](#les-bases-à-avancé-hyper-complet)

## Abréviations

- CPT: Custom Post Type
- CT : Custom Taxonomy

## Must used plugins

Dans le dossier `wp-content/mu-plugins`

- toujours actifs
- utilisateurs ne peuvent pas les désactiver par accident
- chargés avant les plugins normaux

Problèmes

- ils n'apparaissent pas les notifs de mise a jour. C'est à nous de penser à les update
- les hooks d'activation et de desactivation ne sont pas executés pour les must used plugins

En gros, *à utiliser avec parcimonie*. Il vaut mieux y mettre des plugins qui bougent pas trop, autonomes et isolés, surtout pour monitorer le site je dirai.

## CPT(Custom Post Types) et CT (Custom Taxonomies)

Les CPT et CT **sont de préférence déclarés dans un plugin**. Je préfère les mettre dans **`mu-plugins`** pour être sûr qu'ils ne soient pas désactivés par erreur par l'admin.

## Template hierarchy

**Hyper important !**

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
- toujours pas la page listant tous les posts (`mon-site/blog`), soit pour la page 'Posts page'.

En gros c'est pas compliqué : par défaut Wordpress nous met en home la liste des posts (`/` est pareil que `/blog`). Dans ce cas c'est `home.php` qui est appelée. Si on choisit de mettre une page static à la place, il faut aussi choisir une page pour la liste des posts. Et c'est toujours `home.php` qui s'en charge.

- sert de fallback à : front-page 
- fallback : [index](#index.php)

## front-page.php

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

Par défaut, un post standard aura accès à deux types de taxonomy (builtin wp) `Categories` (hierarchical cad que les *terms peuvent avoir aussi des enfants*) and `Tags`(non hierarchical cad que *les terms ne peuvent pas avoir d'enfants*).

Ces deux taxos sont incluses dans Wordpress par défaut. Mais on peut les supprimer si on veut. Y'en a 4 en tout si on compte les `Link Categories` (catégorie pour les liens) et les `Post_Format Categories` (pour les types de post en fonction de leur mimetype).

Les taxonomies sont les parents, les `terms` sont les enfants.

### Taxonomy hierarchy

A lire du haut vers le bas pour voir le flow du templating. Valable pour les taxonomies par défaut (voir la hierarchie général des templates)

~~~
- taxonomy-{taxonomy}-{term}.php
- taxonomy-{taxonomy}.php
- tag-{slug}.php
- tag-{id}.php
- category-{slug}.php
- category-{ID}.php
~~~

Si on l'applique à la taxonomy `Categories` (slug `category`)

~~~
- category-{slug}.php
- category-{id}.php
- category.php
- archive.php
- index.php
~~~

Si on l'applique à la taxonomy `tags` (slug `tag`)

~~~
- tag-{slug}.php
- tag-{id}.php
- tag.php
- archive.php
- index.php
~~~

### Taxonomy custom

On peut créer nos propres taxonomies avec `register_taxonomy( '${slug taxonomy}', array( 'posttype' ), $args )`.

Si on applique la hierarchie à la taxonomy custom 

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

Donc par défaut pour accéder a la liste des posts `uncategorized` l'url sera 

~~~
{`mon-domaine}/category/uncategoried`
~~~

De même, pour accéder a la liste des posts étiquettés `Tags` l'url sera

~~~
{`mon-domaine}/tag/tags`
~~~

On peut changer le slug `category` et `tag` par défaut par un autre dans `Settings/Permalinks/Optional`.

Pour les taxonomies custom. Imaginons on a créé un CPT `projet` et une CT `savoir-faire` avec un terme 'ebenisterie'. Comment lister tous les projets d'ébenisterie ?

~~~
{`mon-domaine}/savoir-faire/ebenisterie`
~~~


### Permalinks (SEO)

Le permalink dans Wordpress c'est l'URL entière d'une page (incluant le protocole, le nom de domain).

Le SEO ou Ranking de vos contenus parmi les résulats retournés par votre navigateur est hyper important si vous voulez être trouvé. Qui va sur la 2eme page de Google resultats ? (Montrer stat)

Le SEO c'est souvent de l'arnaque et de la magie noire (ca depend par exemple de code propriétaire du moteur de Google et les règles peuvent changer très vite) mais c'est important. Et il y a quand même des règles stables dans le temps.

Essayer de prévoir le SEO `avant` de commencer le développement ou la mise en prod de votre site. Cela vous aidera a la structuration de vos pages. Le faire après peut devenir vite compliqué voire impossible (croyez moi).

#### Slug

Les slug sont la dernière partie de l'url, par exemple dans `example.com/about-us` le slug est `about-us`. Dans `example.com/?p=356` il n'y a pas de 

Toujours utiliser `Post name`: plus comphrénsible, SEO friendly

- search engines use your URL to determine what your page or content page is all about. And if they found your permalink related to the content or page, the search engines find it genuine and legitimate to give them priority in their search engine rankings.

- Using SEO keywords in your URL can help you rank for your target keywords. Google uses the URL as a factor in ranking your page, so if the URL slug includes your keywords, then Google will be more likely to rank it.

#### URL best practices

- use HTTPS
- nom de domaine clair et unique
- keep it short and precise (concis). Le poids des derniers mots pour les bots crawlers est plus faible
- lisible pour les robots **et** pour les humains
- utiliser des sous-domaine: `sous-domaine.mon-domaine.com`
- placer des mots clefs qui décrivent le contenu de la page
- eviter de mettre une année dans l'URL
- eviter la duplication d'URL sur votre site
- The best time to change a permalink is *when* you’re originally creating the post content `before` the post is published. Why? If you change a post’s slug after it has already been published, beware that you’re also changing the post’s URL, so the permalink change may create a 404 page. Any links that have been shared online using the old URL will not work any longer and will need to have redirects set up to the new URL. By customizing your post slug prior to publishing the content, you’ll have an optimized permalink the second your hit the publish button.
- consistent structure sur tout le site 


#### Outils SEO Wordpress

 - plugin [Yoast](https://yoast.com/), un des plugins les plus installés et maintenu de l'écosystème de Wordpress. Gratuit possible, user friendly pour vos administrateurs de contenus (clients). Ne pas hésiter à  l'installer.


### Pagination

- [`paginate_links()`](https://developer.wordpress.org/reference/functions/paginate_links/) : on a la main sur le nombre de pages de résultats et le formattage du markup de balisation
- le nombre de posts par page est reglé depuis l'admin dans Settings/Reading
- si on veut faire des choses plus contrôlés on passe par une [WP_Query](https://developer.wordpress.org/reference/classes/wp_query/)

### Divers

- `hierarchical` : 
    - `true`, traite la taxo comme une catégorie (liste d'items prédéfinis),
    - `false` traite la taxo comme un tag. On rentre des termes séparés par des virgules sur chaque post à la volée. 

Si l'on ne souhaite pas avoir une taxonomie hierarchique et que l'on désire quand même avoir les checkboxs (c'est quand même plus cool) on peut utiliser le paramètre `meta_box_cb`

~~~php
$args = array(
		'hierarchical'      => false,
		// utilise les metabox de post comme pour la taxonomy hierarchique 'category'
		'meta_box_cb'       => 'post_categories_meta_box',
	);
register_taxonomy( 'action', array( 'projet' ), $args );
~~~

## template-parts

On crée en général un dossier `template-parts` à la racine du thème. On y stocke tout template html de contenu. Par exemple

- `template-parts/content.php` pour le contenu d'un post
- `template-parts/content-page.php` pour le contenu d'une page
- etc...

Par exemple, le contenu de `content.php`
~~~html
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

On l'appelle ensuite avec la fonction `get_template_part({path du fichier content},{ce qui suit le -}` dans la main loop. Par exemple

~~~php
<?php if (have_posts()) : while (have_posts()) : the_post(); ?>
            <?php get_template_part('template-parts/content','page') ?>
        <?php endwhile; else : ?>
            <?php get_template_part('template-parts/content', 'none') ?>
<?php endif; ?>
~~~

## [Post formats](https://wordpress.org/support/article/post-formats/)

Pour présenter différents types de posts de manière différente. Les formats sont des sortes de métadonnées sur les posts. Par défaut un post a le format `standard`. Utile pour utiliser des template-parts en fonction d'une méta (ex: vidéo, comment etc...)

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

A utiliser pour le format `quote` pour les `testimonials` par exemple, au lieu d'un CPT !


## Sanitization

Process of cleaning or filtering your input data. On parle aussi de sanitazing, validating, filtering, cleaning... Même si on peut dire qu'il y a des différences entre ces termes globalement ils veulent tous dire "preventing invalid from **entering** your application.

Il y a plusieurs approches pour sanitize les données en entrée et certaines sont plus sécurisées que d'autres.

La meilleure approche est de concevoir ce processus comme un processus **d'inspection**. N'essaie pas de corriger des données invalides, force les utilisateurs à jouer selon tes règles. L'histoire a montré que la tentative de corriger des données invalides a mené à des vulnérabilités. 

Avoir une whitelist approach : on fait l'hypothèse que toute donnée qui arrive est invalide sauf si on peut prouver qu'elle ne l'est pas. Ainsi, l'erreur qu'on risque de faire c'est d'invalider une donnée valide. Ce qui est facheux mais moins que l'inverse.

### Méthodologie proposée, exemple en PHP

Un ensemble de données `$data` arrive.

- crée un tableau $clean vide. Ce tableau ne contiendra que des données dont on est sûrs qu'elles ont été validées.
- creer un ensemble de **regles de validation** que $data doit passer avant de se retrouver dans $clean.
- privilégier des fonctions builtin quand c'est possible (plus sur que votre code)

- identifier l'input (donnée considérée comme invalide et sale)
- valider l'input
- distinguer entre donnée sale et donné valide

### Sanitization dans Wordpress

Sanitize strips ou modifie les data, c'est a dire qu'il modifie l'entrée. C'est donc du filtre qui peut ouvrir vers des vulnérabilités. Il faut faire confiance à une fonction maintenue par une communauté depuis 20ans.

- sanitize_text_field
- sanitize_title
- sanitize_email
- sanitize_html_class
- esc_url_raw
- sanitize_user
- sanitize_option
- sanitize_meta

## Escaping (output)

Escaping output : échaper ou encoder des caractères pour être sûrs que leur signification originale est preservée, et eviter d'envoyer un résultat dangereux à un système exterieur à votre site/application (navigateur, base de données, url).

Pareil que pour la sanitization, on fait l'assomption que toute donnée peut etre malveillante (meme si on a validé).

Escaping output est aussi en 3 étaps:

- identifier les output
- échapper les outputs
- distinguer entre donnée échappée et non échappéé

### Pour envoyer des données vers le client (navigateur)

[`htmlentities`](https://www.php.net/manual/fr/function.htmlentities.php) est la meilleure fonction. Prend en entrée une string et retourne une version modifiée de telle sorte que certains caractères ne seront pas intérpretés par le navigateur (balises html ou scrit, "<" ). htmlentities accepte des arguments qui permettent de définir les règles d'échappement et le character set qui doit correspondre à celui indiqué dans le header Content-Type de votre requête réponse au client.

En gros utilisez toujours `htmlentities($donnée_a_echaper, ENT_QUOTES, 'UTF_8')`.

Un exemple

~~~php

//identification de notre sortie
$html=array();

//echappement
$html['username'] = htmlentities($clean['username'], ENT_QUOTES, 'UTF_8');

//servi au client
echo "<p>Welcome back, {$html['username']}.</p>"
~~~

### Pour envoyer des données vers une base de données

 - utiliser une fonction native à votre base de données (implementée pour elle)
 - mysqli::real_escape_string
 - **PDO**, nouveau standard dans PHP. Pour des requetes préparées.

### Escaping dans Wordpress

Wordpress met à disposition tout un tas de fonctions pour l'échappement. Vous n'aurez surement jamais besoin d'utiliser htmlentities directement.

Avant de renvoyer une donnée dans un script php sur la sortie standard, on veut être sûr de pas envoyer du code malfaisant au client, et éviter du cross-site-scripting (XSS). Une [liste de fonctions utils](https://developer.wordpress.org/themes/theme-security/data-sanitization-escaping/#escaping-securing-output)

- `esc_html()`: a utiliser pour recuperer des données dans un élément html

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


## Javascript dependencies

Si on regarde la page du codex de la fonction [wp_enqueue_scripts](https://developer.wordpress.org/reference/functions/wp_enqueue_script/#default-scripts-and-js-libraries-included-and-registered-by-wordpress) on peut voir toutes les dépendences JS du coeur de Wordpress (utilisé côté admin). 

Si on a besoin de l'une de ces librairies **on a pas besoin de les importer explicitement dans notre projet**. Il suffit de copier leur slug dans le tableau `$deps` passé en argument de `wp_enqueue_scripts`.

### Jquery

Si on veut par exemple embarquer `jquery` sur nos pages webs servies au client 

~~~php
wp_enqueue_script( 'main-script', get_stylesheet_directory_uri() . '/js/scripts.min.js', array( 'jquery' ), $js_version, true );
~~~

Dans notre fichier `main-script.js` si on veut utiliser jquery, **il ne faut pas utiliser le signe $ car il peut rentrer en conflit avec d'autres library**.

Donc au lieu d'écrire
~~~javascript
$('.single-projet)
~~~

on écrira

~~~javascript
jQuery('.single-projet)
~~~

## Template tags

Les [`template tags`] sont des fonctions spéciales de wordpress qui permettent d'accéder au contenu/données wordpress (par ex `get_the_title()`, `the_title()`) . Elles passent aussi par des filtres (hook filter) `add_filter('the_title', callback)`.

La plupart de ces fonctions acceptent des paramètres.

~~~php
the_title('<h1>','</h1>');
~~~

## Include tags

- get_header()
- get_footer()
- get_sidebar()
- get_template_part()
- get_search_form()
- comments_template()

## Login tags

- wp_loginout() : pour fair un bouton de connexion/deconnexion facilement. Si on veut être redirigé vers la même page où est présent le bouton on peut faire `<?php wp_loginout( get_permalink() ?>`. La fonction prend en argument une url pour le `redirect` après login ou logout.
- wp_logout_url()
- wp_login_url()
- wp_login_form()
- wp_lostpassword_url()
- wp_register()
- is_user_logged_in()

## Bloginfo tags

- bloginfo(), voir tous les paramètres

## Archive tags

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

## Calendar tags

- **get_calendar()** (pareil que le widget, editable par l'user)
- calendar_week_mod()
- delete_get_calendar_cache()

## Misc tags

- wp_meta()
- get_current_blog_id() (pour le multisites)
- wp_title (deprecated, auto display a present)
- allowed_tags()
- wp_enqueue_scripts()

## Navigation tags

- [wp_nav_menu](https://developer.wordpress.org/reference/functions/wp_nav_menu/): display a menu on the page. Utiliser tous les args pour les wrapper au lieu de faire du markup HTML a la main autour.
- [register_nav_menu](https://developer.wordpress.org/reference/functions/register_nav_menus/): ajouter des locations de menu
- walk_nav_menu_tree (used internally by the core) utilisé pour controler comment les menus sont construits et affichés

## Link tags

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

## Ressources

### Doc officielle wordpress.org

Très bien faite, mais peut parfois demander un peu d'experience pour s'y retrouver 

- [Codex](https://codex.wordpress.org/)
- [Wordpress hierarchy](https://developer.wordpress.org/themes/basics/template-hierarchy/)
- [Wordpress coding standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/)
- [Template tags](https://codex.wordpress.org/Template_Tags)
- [Using Permalinks](https://wordpress.org/support/article/using-permalinks/)

### Articles

#### Taxonomies

- [WordPress Taxonomies: The Ultimate Guide](https://ithemes.com/blog/wordpress-taxonomies)
- [WordPress Permalinks: The Essential Guide](https://ithemes.com/blog/wordpress-permalinks)

### Livres

- *Professional WordPress: Design and Development* de Brad Williams et David Damstra, Edition Wrox, 3rd Edition, 2015
- `Essential PHP Security`, Chris Shiflett, Edition O'REILLY, 2006 (Attention certaines fonctions PHP sont deprecated mais sinon la philosophie et les conseils sont toujours pertinents et bien expliqués. Lecture 'attentive').
- `Modern PHP: new features and goo, Josh Lochart, Edition O'REILLY, 2015

### Podcasts

- https://phproundtable.com/episode/all-things-wordpress
- https://podcast.htmlallthethings.com/e/the-thing-about-wordpress/


- 
### Développement de thèmes

- [Underscores](https://underscores.me/)

### Développement de plugin

- [Plugingplate](https://pluginplate.com/plugin-boilerplate/) : starterpack
- [Introduction](https://developer.wordpress.org/plugins/intro/)

### Formations

- [Zack Gordon (le daron)](https://zacgordon.com/)

#### Gratuit

- [Cours wordpress.org](https://learn.wordpress.org/courses)

#### Payant

##### les bases

- [Become a WordPress Developer: Unlocking Power With Code](https://www.udemy.com/course/become-a-wordpress-developer-php-javascript/), pas mal pour apprendre le processus de la création de thème dans son ensemble

##### les bases à avancé (hyper complet)
- [Complete WordPress Theme & Plugin Development Course [2021]](https://www.udemy.com/course/wordpress-theme-and-plugin-development-course) 
- [WordPress REST API Complete Beginners Guide](https://www.udemy.com/course/wordpress-rest-api-course/)
- [Gutenberg Block Development for WordPress](https://www.udemy.com/course/gutenberg-block-development-wordpress-javascript-course/)


