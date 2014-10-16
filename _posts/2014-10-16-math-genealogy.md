---
layout: post
title: Mathematics Genealogy - Scraping the data
---

# Looking at a mathematicians tree?
 
A fun project that I've been aware of for the past few years is the [Mathematics Genealogy Project](http://genealogy.math.ndsu.nodak.edu/). To quote from their mission statement

<div class="message">
The intent of this project is to compile information about ALL the mathematicians of the world. 
We earnestly solicit information from all schools who participate in the development of research 
level mathematics and from all individuals who may know desired information.
</div>

The type of data that they are storing involves

* The complete name of the degree recipient
* The name of the university which awarded the degree
* The year in which the degree was awarded
* The complete title of the dissertation
* The complete name(s) of the advisor(s)

I've been interested in scraping the data from my [own record in the database](http://genealogy.math.ndsu.nodak.edu/id.php?id=177161) but it's been one of those tasks that I've
kept meaning to get back to. I initially thought I would write a python script to run the scraping myself but after a little google searching I found someone else had already 
presented the world with code to get started!. 

So here I'm going to go through some of the minor changes to the code presented by [Francisco Blanco-Silva](http://blancosilva.wordpress.com/2011/02/14/math-genealogy-project/)
that I had to get make in order to get things running on my machine using Python 3.4.X and offer a little more explanation in places for others that may not be used to playing
around with ugly truth behind webscraping.

## Scraping the advisor data

The original scraper that I tried to get running found [here](http://blancosilva.wordpress.com/2011/02/14/math-genealogy-project/)

```python
import urllib
def augment_genealogy(subject_id,subject_tree):
    # This function assumes that subject_id in not in subject_tree
    f=urllib.urlopen("http://genealogy.math.ndsu.nodak.edu/id.php?id="+subject_id)
    subject=f.read()
    f.close()
    if not subject.count("Advisor: Unknown"):
        # How many advisors did subject have?
        # For each advisor, retrieve their information, and attach
        # them to subject as parents
        advisor_list=[]
        for advisor in range(subject.count("Advisor")):
            subject=subject.partition("Advisor")[2].partition("id=")[2]
        advisor_id = subject[0:subject.index("\"")]
        advisor_list.append(advisor_id)
        if advisor_id not in subject_tree:
            augment_genealogy(advisor_id,subject_tree)
     
    subject_tree[subject_id]=advisor_list
    return subject_tree
    else:
        subject_tree[subject_id]=[]
        return subject_tree
```

threw me a few errors, some of which are due to me running python 3.4.x and some of which looks to have been a copy paste issue! 

First the libraries

```python
import urllib
import pickle # need this to dump the dict file 
```

and here is the augment_genealogy3 function written with the minor adjustments for python 3

```python
def augment_genealogy3(subject_id,subject_tree):
    # This function assumes that subject_id in not in subject_tree
    f=urllib.request.urlopen("http://genealogy.math.ndsu.nodak.edu/id.php?id="+str(subject_id))
    subject=f.read().decode()
    f.close()
    if not subject.count("Advisor: Unknown"):
        # How many advisors did subject have?
        # For each advisor, retrieve their information, and attach
        # them to subject as parents
        advisor_list=[]
        for advisor in range(subject.count("Advisor")):
            subject=subject.partition("Advisor")[2].partition("id=")[2]
            advisor_id = subject[0:subject.index("\"")]
            advisor_list.append(advisor_id)
            if advisor_id not in subject_tree:
                augment_genealogy3(advisor_id,subject_tree)
        
        subject_tree[subject_id]=advisor_list
        return subject_tree
    else:
        subject_tree[subject_id]=[]
        return subject_tree
```

So how about running with and scraping your info? Well we need the `subject_id` of interest; this is just the number that the folks at the MGP have 
assigned your mathematician of interest. When you search for me for example `Sepanda Pouryahya` you should end up with a url that looks like
`http://genealogy.math.ndsu.nodak.edu/id.php?id=177161` and you see my id is `177161`. Now to do the scraping

```python
subject_id = 177161 # my ID
subject_tree={}     # just an empty dictionary 

# Save the data for later use
pickle.dump( subject_tree, open( "math_subject_tree.p", "wb" ) )
```

We now have a dictionary with all the subject's tree. Here's a little snippet of what the data looks like

{'126667': ['125938', '126659'], '125938': [], '127461': ['127275', '127252'], 177161: ['89543'], '125232': [], '134780': ['146365'], '74384': [], '125142': ['127245', '125303'], ...


The dictionary 'keys' are the ID numbers from the website and the value(s) associated with each key contains the ID(s) of the subject's supervisor(s).
Although these records are all mathematicians and the numbers themselves are a pleasure - grabbing 'names' seems like a good idea.


## Associating names with the advisor IDs

Just like the previous scraper [Francisco Blanco-Silva](http://blancosilva.wordpress.com/2011/02/14/math-genealogy-project/)'s blog post has what
I needed - after making a few minor adjustments

Just as a reminder the `subject_tree` data can be loaded up with a call to 

```python
subject_tree = pickle.load( open( "math_subject_tree.p", "rb" ) )
```

To make the dictionary containing the number to name correspondence

```python
#Create a dictionary that maps to each id, its name
names=dict()
for id in subject_tree:
    print(id)
    f=urllib.request.urlopen("http://genealogy.math.ndsu.nodak.edu/id.php?id="+str(id))
    s=f.read().decode()
    f.close()
    name=s.partition("The Mathematics Genealogy Project - ")[2]
    print(name[0:name.index("</title>")])
    names[id] = name[0:name.index("</title>")]

pickle.dump( names, open( "math_subject_tree_names.p", "wb" ) )
```

Now I have all the advisors in my tree - my academic ancestors if you like!

## Future tasks

At some stage I have to put all of this data into a useful looking network graph. 

As it is though, it's nice to see some mathematicians in my academic tree that have caused me untold hours of study and awe.

## The raw list
['Jakob Milich', 'Desiderius Erasmus', 'Ernestus Hettenbach', 'Sepanda Pouryahya', 'Ludolph van Ceulen', 'Demetrios Kydones', 'Abraham Klein (Calovius)', 'Gabriele Falloppio', 'Johannes von Andernach', 'Felix Plater', 'Elissaeus Judaeus', 'Wolfgang Capito', 'Johann von Brunn', 'Gregory Palamas', 'Basilios Bessarion', 'Pietro Roccabonella', 'Simon Grynaeus', 'Heinrich  von Langenstein', 'Janus Lascaris', 'Christian Hausen', 'Johann Segner', 'Georg Calixt', 'Petrus Curtius', 'Nicoletto Vernia', 'Jakob Beuerlin', 'Harold Grad', 'Christoph Meurer', 'Daniel Heffernan', 'Vittore Trincavelli', 'Johann Reuchlin', 'Jan van Campen', 'Bassiano Landi', 'Johann Pfaff', 'Moses Perez', 'Duncan Liddel', 'Demetrios Chalcocondyles', 'François Dubois', 'Johann Bode', 'Rudolf Camerarius', 'Immanuel Tremellius', 'Theodoros Gazes', 'Guillaume Rondelet', 'Salomon Alberti', 'Adolph Vorstius', 'Johannes Caselius', 'Marsilio Ficino', 'Otto Mencke', 'Adriaan van den Spieghel', 'Gottfried Leibniz', 'Richard Liboff', 'Jacob Andreae', 'Johannes Volmar', 'Simon Sulzer', 'Johannes Stupanus', 'Manuel Chrysoloras', 'Georg von Peuerbach', 'Pierre-Simon Laplace', 'Elias Camerarius, Sr.', 'Melchior Jöstel', 'Bonifazius Erasmi', 'Erasmus Reinhold', 'Caspar Peucer', 'Rudolf Lipschitz', 'Balthasar Kaeuffelin', 'Leonhard von Dobschütz', 'Gottfried Möbius', 'Aegidius Strauch', 'Marin Mersenne', 'Johannes Sturmius', 'Richard Courant', 'Angelo Poliziano', 'Jan Jessenius', 'Andreas Vesalius', 'Jean Tagault', 'Leo Outers', 'Cristoforo Landino', 'Rudolf Krause', 'Johann Büsch', 'Jacobus Sylvius', 'Maarten van Dorp', 'Ambrosius Rhodius', 'Johann Wichmannshausen', 'Philipp Melanchthon', 'Petrus Ryff', 'Jacobus Golius', 'David Hilbert', 'Sigismondo Polcastro', 'Valentine Naibod', 'Christoph Notnagel', 'Erhard Weigel', 'Pietro Pomponazzi', 'Johann Planer', 'Jacques Toussain', 'Georgios Gemistos', 'Petrus Ramus', 'Carl Gauß', 'Bruno Zumino', 'Marco Musuro', 'Leonhard Euler', 'Jacobus  Martini', 'Abraham Kästner', 'Rudolph Snellius', 'C. L. Ferdinand Lindemann', 'Adrien Turnèbe', 'Gemma Frisius', 'Martin Ohm', "Jean d'Alembert", 'Girolamo Aleandro', 'Jakob Thomasius', 'Nikolaus Eglinger', 'Julius Plücker', 'Christian Gerling', 'Johann Hoffmann', 'Philipp Müller', 'Nilos Kabasilas', 'Georg Metzger', 'Willebrord Snellius', 'Guillaume Budé', 'Valentin Thau', 'Jacob Bernoulli', 'Gaetano da Thiene', 'Theodor Zwinger, Jr.', 'Bartholomäus Schwendendörffer', 'Theodore Metochites', 'Manuel Bryennios', 'Gilbert Jacchaeus', 'Antonio Brasavola', 'Giovanni Battista della Monte', 'Jean-Baptiste Fourier', 'Nicolas Clénard', 'Johann Pasch', 'Frans van Schooten, Jr.', 'Johannes  von Gmunden', 'Johannes Hommel', 'Werner Rolfinck', 'Joseph Lagrange', 'Nicolaus Copernicus', ' Pelope', 'Joseph Scaliger', 'Emmanuel Stupanus', 'Michael Walther, Jr.', 'Ulrich Zasius', 'Cornelius Martini', 'Niccolò Leoniceno', 'Johann  Bauhin', 'Andreas Schato', 'Nicolas Malebranche', 'Domenico Novara da Ferrara', 'Hieronymus Fabricius', 'Karl von Langsdorf', 'Friedrich Leibniz', 'Johann Grynaeus', 'Simeon Poisson', 'Franciscus Sylvius', 'Jan Stampioen, Jr.', 'Sebastian Beck', 'Georg Hamberger', 'Georg Joachim Rheticus', 'Peter Werenfels', 'Gustav Dirichlet', 'Christiaan Huygens', 'Simon Hilscher', 'Valentin Otto', 'Moritz Steinmetz', 'Ognibene Bonisoli da Lonigo', 'Jacobus Latomus', 'Scipione Fortiguerra', 'Theodor Zwinger', 'Guarino da Verona', 'Rutger Rescius', 'Jacob ben Jehiel Loans', 'Georgius Hermonymus', 'Sebastian Dietrich', 'Johann Wedel', 'John Craig', 'Vittorino da Feltre', 'Matthaeus Adrianus', 'Johannes Stöffler', 'Johann Bernoulli', 'C. Felix Klein', 'Thomas Erpenius', 'Daniel Sennert', 'Johannes Argyropoulos', 'Paul Wittich', 'Giovanni Battista Beccaria', 'Andreas Kunad', 'Matteo Realdo Colombo', 'Georg Wedel', ' Luca Pacioli', 'Johann Quenstedt', 'Johannes Regiomontanus']




