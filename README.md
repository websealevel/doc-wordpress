# doc-wordpress

[TOC]

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

- ne jamais utiliser ${id} comme template ! Sauf en cas de force majeure. Trop dépendant de la bdd et peu explicite



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

## Escaping output

### Sécurité

Avant de renvoyer une donnée dans un script php sur la sortie standard, on veut être sûr de pas envoyer du code malfaisant au client, et éviter du cross-site-scripting (XSS). Une [liste de fonctions utils](https://developer.wordpress.org/themes/theme-security/data-sanitization-escaping/#escaping-securing-output)

- `esc_html()`: a utiliser pour print des données dans un élément html
- etc...

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

Les `template tags` sont des fonctions spéciales de wordpress qui permettent d'accéder au contenu/données wordpress (par ex `get_the_title()`, `the_title()`) . Elles passent aussi par des filtres (hook filter) `add_filter('the_title', callback)`.

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

Listings

- single_post_title(), outside of the loop
- post_type_archive()
- single_cat_title()
- single_tag_title()
- single_term_title()
- 

## Ressources

### Doc officielle

- [Codex](https://codex.wordpress.org/)
- [Wordpress hierarchy](https://developer.wordpress.org/themes/basics/template-hierarchy/)
- [Wordpress coding standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/)
- [Template tags](https://codex.wordpress.org/Template_Tags)

### Articles

#### Taxonomies

- [WordPress Taxonomies: The Ultimate Guide](https://ithemes.com/blog/wordpress-taxonomies)

### Livres

- *Professional WordPress: Design and Development* de Brad Williams et David Damstra, Edition Wrox, 3rd Edition (27 janvier 2015)

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


