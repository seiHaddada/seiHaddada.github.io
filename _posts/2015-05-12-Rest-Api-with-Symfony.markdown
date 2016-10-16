---
layout: post
title:  "Rest Api with Symfony !"
date:   2015-05-12 20:53:51:02
categories: Rest Symfony thoughts
comments: True
---
## **Introduction:**
Developing a REST API is not an easy task because it’s hard to make changes once it’s released and 
It takes so much time designing the api.  

## **Case study:**
---

In my study project I have to develop a multi-platform application (a desktop, mobile, and web)
So I preferred to use a REST approach to facilitate the work.

Those blog posts helped me to get through the setting up of the API.  
Martin flower [described][described] the model very well .  
Since I am going to use Symofny framework wiliam daurad’s [blog post][blog] was very helpful.

## **Pre requires:**
---

The objective here is to create a symfony Bundle inside my application that serves api for Offre
(it’s written in French it means Offer) with get and post using symfony2 [FOSRestBundle][FOSRestBundle], the [NelmioApiDocBundle][NelmioApiDocBundle], the [JSMSerializerBundle][JSMSerializerBundle], and [Doctrine][Doctrine].

{% highlight json %}
/*composer.json*/
        "friendsofsymfony/rest-bundle": "1.*",
        "jms/serializer-bundle": "0.13.*",
        "nelmio/api-doc-bundle": "~2.8",
{%endhighlight%}
appKerlnel.php
{% highlight ruby %}
/*appKernel.php*/
            new FOS\RestBundle\FOSRestBundle(),
            new JMS\SerializerBundle\JMSSerializerBundle(),
            new Nelmio\ApiDocBundle\NelmioApiDocBundle(),
{% endhighlight %}

###**Step 1 :The entity**
---
This is my already implemented Offre entity 

{% highlight php  %} <?php
class Offre
{
    /**
     * @var integer
     *
     * @ORM\Column(name="id", type="integer", nullable=false)
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="IDENTITY")
     */
    private $id;

    /**
     * @var string
     *
     * @ORM\Column(name="etat", type="string", length=20, nullable=false)
     */
    private $etat;

    /**
     * @var string
     *
     * @ORM\Column(name="typeImmob", type="string", length=20, nullable=false)
     */
    private $typeimmob;
    /** getters and setters*/
}?>
{% endhighlight %}
actually we will not create a form for this entity we don't recommand this but it is a choice for our project.  

###**Step 2: Offre Handler**
---
{%highlight php%}
<?php
class offreHandler 
{

    private $om;
    private $offreClass;
    private $repository;

    // ..
    public function __construct(ObjectManager $om, $offreClass)
    {
        $this->om = $om;
        $this->offreClass = $offreClass;
        $this->repository = $this->om->getRepository($this->offreClass);
    
    }
?>
{%endhighlight%}

this handler will serve the offres with  `get($id)` to get a single offre and `getAll($limit,$offset)` for all offres with limit and offset parameters.  
`post(Request $parameters)` to post offres.

{%highlight php%}
<?php
 public function getAll($limit = 5, $offset = 0)
    {
        return $this->repository->findBy(array(), null, $limit, $offset);
    }

 public function get($id)
    {
        return $this->repository->find($id);
    }
 
 public function post(Request $parameters)
    {
        $offre = new Offre(); // factory method create an empty Offre

        return $this->processPost($offre, $parameters, 'POST');
    }

?>
{%endhighlight%}

processPost will do every thing, getting the attributes values and returning our offre object. like i said before we didn't use the Form so we will get this attribute by attribute.
{%highlight php%}
<?php
private function processPost(Offre $offre, Request $request, $method = "POST")
    {
            
            $etat=$request->get('etat');
            $typeimmob=$request->get('typeimmob');
            //we should also do the validation of our attributes
            $offre->setEtat($etat);
            $offre->setTypeimmob($typeimmob);
            $this->om->persist($offre);
            $this->om->flush($offre);
            return $offre;
    }
?>
{%endhighlight%}

###**Step 3 :The Controller**
---
now after implementing our handler our work is to call those methodes in our offreController 

{%highlight php%}
<?
  public function getOffresAction(Request $request, ParamFetcherInterface $paramFetcher)
    {
        $offset = $paramFetcher->get('offset');
        $offset = null == $offset ? 0 : $offset;
        $limit = $paramFetcher->get('limit');

       $entities= $this->container
        ->get('rest.offre.handler')
        ->getAll($limit, $offset);

        $view = $this->view($entities, 200)
          ->setTemplate("sprint2restBundle:Default:index.html.twig")
          ->setTemplateVar('entities') ;

        return $this->handleView($view);
    }
?>
{%endhighlight%}
what is `rest.offre.handler` ??  
rest.offre.handler is a service it will help us to access  the handler and use its functionality wherever you need.

*Service Container:*
{%highlight xml%}
 <parameters>
        
        <parameter key="rest.offre.handler.class">sprint2\restBundle\Handler\offreHandler</parameter>
</parameters>
 <services>
        <service id="rest.offre.handler" class="%rest.offre.handler.class%">
            <argument type="service" id="doctrine.orm.entity_manager" />
            <argument>%rest.offre.class%</argument>
        </service> 
</services>
{%endhighlight%}
for more documentation for the service container here [the official symfony article][article]  
now let our api speak , we will use nemlio for that.
{%highlight php%}
<?
    /**
     * Get All Offres,
     *
     * @ApiDoc(
     *   resource = true,
     *   description = "Gets All offres ",
     *   
     *   statusCodes = {
     *     200 = "Returned when successful",
     *    
     *   }
     * )
     *
     *@Annotations\View(templateVar="entities")
     *@Annotations\QueryParam(name="offset", requirements="\d+", nullable=true, description="Offset from which to start listing offres.")
     *@Annotations\QueryParam(name="limit", requirements="\d+", default="5", description="How many offres to return.")
     * 
     * 
     *
     * @param Request               $request      the request object
     * @param ParamFetcherInterface $paramFetcher param fetcher service
     *
     * 
     * @return array
     **/
    public function getOffresAction(Request $request, ParamFetcherInterface $paramFetcher){...}
    ?>
{%endhighlight%}  

before going further we should see our work is going in the right direction , this is the routing.yml

{%highlight yaml%}
sprint2rest_getOffres:
    type: rest
    prefix: /v1
    resource: sprint2\restBundle\Controller\offreRestController
{%endhighlight%} 

this should generate a route for us `/v1/offres.{_format}` to see this hit  `app/console router:debug | grep v1` you could access the api with `curl -s` or you can download [chrome extension for rest api client][extension] or simply you can use the http command `http localhost:8000/v1/offres.json`

##**The result:**
---
debuging the router:
![debuging the route]({{ site.url }}/images/routingDebug.png)


getting the result with http 
![http]({{ site.url }}/images/http.png)

Documentation:
![doc]({{ site.url }}/images/documentation.png)
![doc1]({{ site.url }}/images/documentation1.png)




[article]:http://symfony.com/doc/current/book/service_container.html
[extension]:https://chrome.google.com/webstore/detail/advanced-rest-client/hgmloofddffdnphfgcellkdfbfbjeloo?hl=fr
[described]:http://martinfowler.com/articles/richardsonMaturityModel.html
[blog]:http://williamdurand.fr/2012/08/02/rest-apis-with-symfony2-the-right-way/
[FOSRestBundle]:https://github.com/FriendsOfSymfony/FOSRestBundle
[NelmioApiDocBundle]:https://github.com/nelmio/NelmioApiDocBundle
[JSMSerializerBundle]:https://github.com/schmittjoh/JMSSerializerBundle
[Doctrine]:http://www.doctrine-project.org/