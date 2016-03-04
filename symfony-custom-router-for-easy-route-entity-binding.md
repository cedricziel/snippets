# Custom Router for Symfony 2 Applications that resolves entities

**Challenge:** I have to be very specific when generating routes with the twig
`path` helper with the stock router. So for example I have a `Blog` entity:

```php
/**
 * Blog
 *
 * @ORM\Table(name="blog")
 */
class Blog {

    /**
     * @var int
     *
     * @ORM\Column(name="id", type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue  (strategy="AUTO")
     */
    private $id;
    
    /**
     * @var string
     *
     * @ORM\Column(name="title", type="string", length=255)
     */
    private $title;
    
    /**
     * @var string
     *
     * @ORM\Column(name="slug", type="string", length=255, unique=true)
     * @Gedmo\Slug(fields={"title"})
     */
    private $slug;
    
    // Getters, Setters...
}
```

The entity already generates slugs from the `title` field into the `slug` field through doctrine extensions.
So now when generating URLs in the templates, I have to either address the id field or the slug field, but I still
have to be explicit about which field needs to be used:

```twig
{{ path('app_blogs_show', {'id': blog.id}) }} -> /blogs/3
{# or the following, depending on the name of the route parameters #}
{{ path('app_blogs_show', {'slug': blog.slug}) }} -> /blogs/using-the-slug-cool-eh
```

But something I could never do is just throwing the object in the path function
and have it generate the URL according to the field you want to auto-reference - like a slug instead of an ID
is not possible:

```twig
{{ path('app_blogs_show', {'blog': blog}) }} // Exception
```

Say we have the following routable controller:

```php
/**
 * Activity controller.
 * @Route(path="/{category}/blogs")
 */
class ActivityController extends Controller
{
    /**
     * Finds and displays a Blog entity.
     * @ParamConverter(name="category", class="AppBundle:Category", options={"mapping": {"category": "slug"}})
     * @ParamConverter(name="blog", class="AppBundle:Blog", options={"mapping": {"blog": "slug"}})
     * @Route("/{blog}")
     * @Method("GET")
     * @Template()
     *
     * @param Category $city
     * @param Blog $blog
     *
     * @return \Symfony\Component\HttpFoundation\Response
     */
     public function showAction(Category $category, Blog $blog){}
}
```

Route generation should be as easy as calling `path('app_blogs_show', {'category': category, 'blog': blog})` and generate
something like `/random-things/blogs/famous-hackernews-article`.

That's (AFAIK, I like to be corrected) only possible when I interchange the default router in the container and extract the
parameter there.

A very simple example would be to have all your entities implement a simple marker-interface so we can decide which types of
objects we want to operate on.

`AppBundle/Entity/RoutableEntity.php`
```php
<?php
namespace AppBundle\Entity;

/**
 * Of course you could enforce a convention here:
 * `getRouteKey` & `getRouteKeyName` or the like
 */
interface RoutableEntity{}
```

Custom router that resolves the correct fields (I18nRouter is used as base but the Stock router
can be extended as well):

`AppBundle/Routing/Router.php`
```php
<?php

namespace AppBundle\Routing;

use AppBundle\Entity\RoutableEntity;
use JMS\I18nRoutingBundle\Router\I18nRouter;

class Router extends I18nRouter
{
    /**
     * Converts doctrine entities so they can be used as arguments to the
     * route generator.
     *
     * @param string $name
     * @param array $parameters
     * @param int $referenceType
     * @return string
     */
    public function generate($name, $parameters = array(), $referenceType = self::ABSOLUTE_PATH)
    {
        foreach($parameters as $paramName => $value) {
            if ($value instanceof RoutableEntity) {
                if (method_exists($value, 'getSlug')) {
                    $parameters[$paramName] = $value->getSlug();
                } elseif (method_exists($value, 'getId')){
                    $parameters[$paramName] = $value->getId();
                }
            }
        }
        return parent::generate($name, $parameters, $referenceType);
    }
}
```

Then register it as a service and replace it in the container with a compiler pass:

`AppBundle/Resources/config/services.yml`
```yml
services:
    app.router:
        class: AppBundle\Routing\Router
        parent: jms_i18n_routing.router
        public: false
```

`AppBundle/DependencyInjection/Compiler/ReplaceRouterCompilerPass.php`
```php
namespace AppBundle\DependencyInjection\Compiler;

use Symfony\Component\DependencyInjection\Compiler\CompilerPassInterface;
use Symfony\Component\DependencyInjection\ContainerBuilder;

/**
 * Class ReplaceRouterCompilerPass
 * @package AppBundle\DependencyInjection\Compiler
 */
class ReplaceRouterCompilerPass implements CompilerPassInterface
{

    /**
     * You can modify the container here before it is dumped to PHP code.
     *
     * @param ContainerBuilder $container
     */
    public function process(ContainerBuilder $container)
    {
        $container->setAlias('router', 'app.router');
    }
}
```
