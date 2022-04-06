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

### Commentaires

- ne jamais utiliser ${id} comme template ! Sauf en cas de force majeure. Trop dépendant de la bdd et peu explicite


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


## Ressources

### Livres

- *Professional WordPress: Design and Development* de Brad Williams et David Damstra, Edition Wrox, 3rd Edition (27 janvier 2015)

### Podcasts

- https://phproundtable.com/episode/all-things-wordpress
- https://podcast.htmlallthethings.com/e/the-thing-about-wordpress/

### Doc officielle

- [Codex](https://codex.wordpress.org/)
- [Wordpress hierarchy](https://developer.wordpress.org/themes/basics/template-hierarchy/)
- [Wordpress coding standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/)
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


