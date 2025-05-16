# Projet-MSBI-SSIS-ETL-
Projet MSBI SSIS (ETL)


Projet MSBI SSIS

Signification des termes :

 - MSBI : Microsoft Business Intelligence
 - SSIS : SQL Server Integration Services
 - SSAS : SQL Server Analysis Services
 - SSRS : SQL Server Reporting Services
 - SSMS : SQL Server Management Studio
 - ETL : Extract, Transform, Load

L’objectif général de ce projet est l’analyse du chiffre d’affaires par produit, par catégorie, par employé, par client, par pays et par région.
Ce projet est en réalité subdivisé en trois grandes parties : la partie SSIS, la partie SSAS et la partie SSRS.

Nous ne nous intéresserons dans le présent travail qu’à la partie SSIS. Dans cette partie, où nous réaliserons les processus ETL (Extract, Transform, Load), l’objectif principal sera la création d’une base de données Data Warehouse (DWH).

Dans le présent travail :

- On utilise un Serveur SQL Azure qui va héberger toutes nos bases de données (nom du serveur : luigysql — le mien).
- On utilise également SSMS (SQL Server Management Studio), une interface graphique qui permet de se connecter à un serveur SQL Azure (ou local), de gérer les bases de données qu’il contient, d’en créer d’autres, d’écrire et d’exécuter des requêtes SQL, de créer des procédures, des vues, etc.
- Les processus ETL sont réalisés dans Visual Studio Community grâce au package SSIS.

Nous partons d’une base de données source appelée "maBaseSQL1", qui est en fait la même que la base de données "TSQL2012" disponible en ligne.
Cette base de données contient plusieurs tables, dont : Commande, Ligne de commande, Client, Employé, Produit, Catégorie, etc.

À partir des 6 tables précédentes, nous créerons une base de données "ODS" qui contiendra 4 tables :

- Une table Commande, obtenue par jointure des tables Commande et Ligne de commande de la base de données "maBaseSQL1".
- Une table Produit, obtenue par jointure des tables Produit et Catégorie de la base de données "maBaseSQL1".
- Une table Client, directement issue de la base de données "maBaseSQL1".
- Une table Employé, directement issue de la base de données "maBaseSQL1".

Ensuite, nous nous baserons sur la base de données "ODS" pour créer la base de données "Data Warehouse (DWH)", qui contiendra les tables FactCommande, dimProduit, dimClient, dimEmployé.
Les trois tables de dimension précédentes sont en fait créées simplement à partir des tables Produit, Client et Employé de la base de données ODS.
La table FactCommande, quant à elle, contient les colonnes relatives aux clés primaires de nos trois tables de dimensions, ainsi que la colonne chiffre d’affaires de la table Commande présente dans la base ODS.

Avant de commencer, récapitulons ce qui a été dit précédemment :

- Nous avons trois bases de données au total, créées dans le serveur SQL Azure via SSMS (plus complexe...).
- Nous avons une base de données source nommée "maBaseSQL1", qui contient toutes nos données.
- Nous avons les bases de données "ODS" et "DWH", qui sont pour l’instant vides mais seront remplies comme décrit plus haut, par le biais des processus ETL.

Pour remplir les bases de données "ODS" et "DWH", nous utilisons SSIS via Visual Studio Community.

Pour cela, commençons par ouvrir Visual Studio Community puis créer un projet SSIS.

############################################################ Gestionnaires de connexions ########################################################

Une fois le projet SSIS ouvert, nous commençons par établir une chaîne de connexion entre nos différentes bases de données : "maBaseSQL1", "ODS" et "DWH".
Pour cela, nous allons dans Gestionnaire de connexions, puis nous choisissons comme type de connexion "Microsoft OLEDB Driver for SQL Server". Ensuite, nous nous connectons l'une après l'autre à chacune des trois bases de données. Après cela, nous les intégrons les unes à la suite des autres.

Maintenant, nous allons créer dans Packages SSIS nos différents packages.

########################################################### Packages ODS #################################################################

Nous commençons par créer le package nommé commande_ods, qui représente la construction de la table Commande qui sera chargée dans la base de données "ODS".
Dans le même état d'esprit, nous créons les packages client_ods, produit_ods et employe_ods.
Ensuite, pour chaque package, nous créerons une tâche de flux de données qui servira à faire l’échange des données entre la table source et la table de destination.

########################################################## Tâche de flux de données ODS #######################################################

- Nous commençons par créer une tâche de flux de données pour le package commande_ods.

- Nous appelons cette dernière "chargement de la table commande", puis nous double-cliquons dessus avant de choisir la source et la destination OLE DB pour faire l’échange.

- On nomme la source "commande_maBaseSQL1" et la destination "commande_ods".

- Nous double-cliquons sur la source "commande_maBaseSQL1" et nous choisissons la base de données source "maBaseSQL1".

- On choisit comme mode d’accès aux données "commande SQL", car la table Commande créée dans "ODS" sera une jointure des tables Commande et Ligne de commande de "maBaseSQL1".

- On fait générer la requête, puis on ajoute une table.

- On ajoute les deux tables "order" (Commande) et "orderDetails" (Ligne de Commande).

- On sélectionne dans les deux tables les colonnes qu’on veut voir apparaître dans notre table finale Commande (on constate qu’il crée en même temps notre requête).

- En plus de la jointure avec les colonnes sélectionnées, nous voulons ajouter une colonne supplémentaire relative au chiffre d’affaires. Pour cela, nous allons modifier la requête qu’il a générée.

- On ajoute donc cette partie :
(Sales.OrderDetails.unitprice * Sales.OrderDetails.qty ) * (1 - Sales.OrderDetails.discount ) à la requête.

- On va dans Colonne et on renomme les colonnes de la table de sortie.

- On valide et on joint les commandes cibles et sources, ensuite on paramètre la commande cible, on valide le tout, et on voit dans SSMS que notre table Commande a bien été créée dans "ODS", mais pour l’instant, elle est vide. Lorsque nous allons exécuter notre package dans SSIS, il va la remplir. On peut ensuite le vérifier dans SSMS.

- On ajoute une tâche d’exécution de requête à notre tâche de flux de données pour que, si jamais un utilisateur modifiait un élément dans notre base de données source "maBaseSQL1", la base de données "ODS" ne soit pas modifiée.

Maintenant, on va créer les packages des autres tables pour la même base de données "ODS".

Après avoir fini cela, on passe maintenant à la création des packages pour la base de données "Data Warehouse" (DWH).

############################################### Packages, Tâche de flux de données, dimension à variable lente, recherche - DWH #######################################

Cette sous-section ne sera pas du tout détaillée, elle est très longue. Nous y disons simplement ce que nous y avons fait.
Nous avons créé les tables de dimension relatives à la base de données DWH.
Nous avons créé trois tables de dimension : dimClient, dimProduit, dimEmploye, et une table FactCommande.
Les trois tables de dimension précédentes sont en fait créées en utilisant simplement les tables Produit, Client et Employé de la base de données ODS.
La table FactCommande, quant à elle, contient les colonnes relatives aux clés primaires de nos trois tables de dimension, ainsi que la colonne chiffre d’affaires de la table Commande qui est dans la base de données ODS.

L’ensemble des résultats peut être visualisé ci-dessus.
