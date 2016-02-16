Acceptance Test Driven Development with Cookiecutter-Django and Hitch Test
==========================================================================

Cookiecutter-Django
-------------------

Set up::

    $ curl https://raw.githubusercontent.com/mitsuhiko/pipsi/master/get-pipsi.py | python

    # It looks like ~/.local/bin/ is not on your PATH?

    $ echo PATH=~/.local/bin/:$PATH >> ~/.bashrc

    $ pipsi install cookiecutter

    $ cookiecutter https://github.com/crdoconnor/cookiecutter-django.git
    
    [ Answer questions ]


Hitch test
----------

Set up and run example test::

    $ cd tests/

    $ pipsi install hitch

    $ hitch init

    $ hitch test register-and-log-in.test

    
Create test
-----------

Put into catporn/tests/sign-up-log-in-and-add-cat.test::

    - name: Sign up, log in and add cat
      scenario:
        - Load website
        - Click: sign-up-link
        - Fill form:
            id_username: testuser
            id_email: testuser@domain.com
            id_password1: password
            id_password2: password
        - Click: sign-up-button
        - Wait for email:
            containing: Please Confirm Your E-mail Address
        - Click on link in last email
        - Click: confirm-button
        - Fill form:
            id_login: testuser
            id_password: password
        - Click: sign-in-button
        - Click: add-cat-link
        - Fill form:
            id_cat_name: Babushka Cats
            id_cat_url: "http://www.catster.com/wp-content/uploads/2015/06/babushka.jpg"
        - Click: add-cat-button
        - Wait for picture to appear:
            item: cat-picture
            containing: "http://www.catster.com/wp-content/uploads/2015/06/babushka.jpg"


Run the test (to see it fail)::

    $ hitch test register-log-in-and-add-cat.test


Make test run
-------------

Add link to user_detail.html page::

    <a id="add-cat-link" class="btn btn-primary" href="{% url 'add_cat' %}">Add cat</a>
    
Add URL to urls.py::
    
    url(r'^addcat/$', add_cat),

Add import to urls.py::

    from catporn.views import add_cat
    
Add app to INSTALLED_APPS::
    
    'catporn',

Add views.py::

    # -*- coding: utf-8 -*-
    from django.contrib.auth.decorators import login_required
    from catporn.forms import AddCatForm
    from catporn.models import Cat
    from django.http import HttpResponseRedirect
    from django.shortcuts import render


    @login_required
    def add_cat(request):
        """Add cat."""
        if request.method == 'POST':
            form = AddCatForm(request.POST)
            
            if form.is_valid():
                Cat.objects.create(
                    url=form.cleaned_data['cat_url'],
                    name=form.cleaned_data['cat_name'],
                    user=request.user,
                )
                return HttpResponseRedirect('/')
        else:
            form = AddCatForm()
        return render(request, 'add_cat.html', {'form': form})

Add form::

    from django import forms

    class AddCatForm(forms.Form):
        cat_url = forms.CharField(max_length=300, label='Url')
        cat_name = forms.CharField(max_length=100, label='Cat pic name')
        

Add model::

    # -*- coding: utf-8 -*-
    from __future__ import unicode_literals, absolute_import

    from django.utils.translation import ugettext_lazy as _
    from django.core.urlresolvers import reverse
    from django.db import models
    from django.conf import settings


    class Cat(models.Model):
        """Any kind of thing that will be bought and sold."""

        name = models.CharField(_("Name"), max_length=255)
        url = models.CharField(_("Url"), max_length=255)
        user = models.ForeignKey(settings.AUTH_USER_MODEL)


        def __str__(self):
            return self.name

        def get_absolute_url(self):
            return reverse('catporn:cat_url', kwargs={'id': self.id})

Add migrations directory::

    $ mkdir catporn/migrations
    $ touch catporn/migrations/__init__.py
        
Also run migrations::

    In [4]: self.services['Django'].manage("makemigrations").run()

Replace home page URL::

    from catporn.views import add_cat, home_page

    url(r'^$', home_page, name="home"),
    
Add home page view to views::

    def home_page(request):
        """Show cats."""
        return render(
            request,
            'pages/home.html',
            {'cats': Cat.objects.all(), }
        )

Override pages/home.html::

    {% extends "base.html" %}
    {% block content %}

    <div class="container">

    <div class="row">
        <div class="col-sm-12">
            <h2>Cats</h2>

        {% for cat in cats %}
            <img class="cat-picture" src="{{ cat.url }}" />
        {% endfor %}
        </div>
    </div>
    </div>
    {% endblock content %}

Add step to detect cat picture in tests/engine.py::

    def detect_picture(self, which, index, url):
        """Detect that picture contains the correct URL."""
        assert url == self.driver.find_elements_by_class_name("cat-picture")[0].get_attribute("src")