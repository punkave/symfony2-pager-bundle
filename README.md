symfony2-doctrine-pager
=======================

Introduction
============

While developing a search page using a very much custom Doctrine2 ORM QueryBuilder, I ran into a snag when trying to display every search result simultaneously. The solution was obviously pagination. Unfortunately, we couldn't use the SonataAdmin pager because there would be too much overhead for such a simple problem. This bundle provides a pager service that simply takes a route (with parameters), and a Doctrine QueryBuilder object. It produces simple pagination links for the entire result set of the query. I have also provided a simple twig template that can be included in any search page so long as the pager has been passed to the view layer.

Requirements
============

* Symfony2
* Doctrine2
* Twig
* Bootstrap (optional)

Installation
============

1) Add the following line to your Symfony2 deps file:
    
    [PagerBundle]
        git=https://github.com/punkave/symfony2-pager-bundle.git
        target=/bundles/PunkAve/PagerBundle

2) Modify your AppKernel with the following line:

    new PunkAve\PagerBundle\PunkAvePagerBundle(),

3) Add the following line to your autoload.php file:

    'PunkAve' => __DIR__.'/../vendor/bundles',

4) Install your vendors

    bin/vendors install

Usage
=====

1) Simply get the pager service from the DIC. If no type of pager is specified, DoctrineORM pager is supplied

	$pager = $this->get('punk_ave.pager_factory')->createPager("DoctrineORM");

2a) Bind the pager to the current request to set the current page and the current route:

    $request = $this->getRequest();
    $pager->bindRequest($request);

    WARNING: if you are using "forward()" rather than "redirect()" to pass a request to a different action, you
    must specify the route explicitly, as the request has no _route property after a forward(). See 
    2b) for how to do this. You can still use bindRequest() to take care of the page parameter.

2b) Optionally pass the current page to the pager and set the route (With params) on the pager:

	$currentPage = ($request->query->has('page'))? $request->query->get('page') : 0;
    $pager->setCurrentPage($currentPage);
    $pager->setRoute($this->getRequest()->get('_route'), $this->getRequest()->query->all());

3) Pass the QueryBuilder you have setup to the pager:

	$pager->setQueryBuilder($queryBuilder);

The pager will automatically clone your query builder to carry out a separate query to count all
rows. With the DoctrineORM pager, this is done by replacing the SELECT cause of your query builder
with a simple count of distinct ids. This is very efficient because it avoids fetching all rows.
Your JOINs, WHERE clauses, etc. remain intact, so in most cases this is sufficient. However you might 
not have a primary key called id, or a SQL error may occur because you have custom aliases in the SELECT 
clause of your query that are essential to other clauses of the query. In such cases, just create 
your own query builder that returns a count of rows and pass it as a second argument to setQueryBuilder:

    $pager->setQueryBuilder($queryBuilder, $countQueryBuilder);

If it is simply not possible to construct a query builder object that returns the count of rows, you
can still work around it! Count the rows yourself and call:
    
    $pager->setNumResults($n)

Hint: when you can't retrieve the number of rows directly, you can still retrieve just
the IDs and other aliases essential to your query, use getScalarResult() to avoid creating objects,
and count() the result of that call. This is generally very fast, even for thousands of items. For
millions of items, consider using Doctrine 2.2's new Paginator object instead.

4) Pass the pager to the view:

	return $this->render('Bundle:Module:view.html.twig', array(
            'pager' => $pager
        ));

To get the results simply call:

	$pager->getResults();

To get an associative array of page links, simply call:

	$pager->getPageLinks();

5) Optionally include the provided pager view in your action's view:

    {% include 'PunkAvePagerBundle:Pager:pager.html.twig' %}

The pager has been styled using Bootstrap conventions. If you have Bootstrap in your project, the pager should look pretty!
