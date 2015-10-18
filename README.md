# tutoriel-spring-batch
Spring batch  Tuto : http://jeremy-jeanne.developpez.com/tutoriels/spring/spring-batch/

Spring batch

Tuto : http://jeremy-jeanne.developpez.com/tutoriels/spring/spring-batch/

- L'architecture de Spring-Batch est constituée de deux couches :
    - la couche Batch Core
    - la couche Batch Infrastructure.

- La couche « Batch Core » contient une API permettant de lancer, monitorer et de gérer un batch. 
Les principales interfaces et classes que contient l'API sont : Job, JobLauncher et Step. 
Schématiquement un batch correspond à un job qui va être lancé via un JobLauncher.

- Un job est constitué d'un ou plusieurs Steps. Un Step correspond à une étape dans un batch.

- La couche « Batch Infrastructure » contient une API fournissant comme principales interfaces : ItemReader, ItemProcessor et ItemWriter. 

Projet :
- créer une bdd batch_tuto en mysql
- créer la table PERSONNE

- création projet maven dans eclipse :
	File > New > Maven Project > Create a simple maven project (skip archetype selection)
	Renseigner <groupId>com.rija.dev</groupId>
	<artifactId>tutoriel-spring-batch</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>tutoriel-spring-batch</name>
	<description>Tutoriel spring batch</description>
	
- ajouter dépendances maven
spring-batch-infrastructure, spring-batch-core, spring

- créer Bean Personne dans com.rija.dev

- créer le fichier de configuration spring batch /tutoriel-spring-batch/src/main/resources/batch-context.xml

- ajouter l'ItemReader : 
	- Spring Batch fournit des implémentations de l'interface ItemReader notamment pour parser des fichiers plats et des fichiers XML.
	- créer le bean personneReaderCSV dans batch-context.xml

- ajouter l'ItemProcessor : C'est dans l'ItemProcessor qu'il y aura la place pour implémenter les règles de gestion du batch. (si la civilite a la valeur M la personne sera ecrite en base sinon on la rejette) 
	- créer la class PersonProcessor dans com.rija.dev
	- Ajouter le bean personProcessor dans batch-context.xml

- ajouter l'ItemWriter : L'ItemWriter va pérenniser les données qui ont été traitées via l'ItemProcessor. Dans cet, sauvegarder les données en base de données dans la table PERSONNE, en utilisant le template jdbcTemplate fourni par Spring.
	- Créer la class PersonJdbcWriter dans com.rija.dev
	- Ajouter le bean personDaoWriter dans batch-context.xml
	
- création du job dans batch-context.xml
	<job id="importPersonnes" xmlns="http://www.springframework.org/schema/batch">
		<step id="readWritePersonne">
			<tasklet>
				<chunk reader="personneReaderCSV" writer="personDaoWriter" processor="personProcessor" commit-interval="2" />
			</tasklet>
		</step>
	</job>
	- Le job sera donc constitué d'un seul Step dont la tasklet est de type « Chunk ».

	- Le chunk est un concept important dans Spring Batch qui définit l'enchainement qui va s'exécuter dans une transaction.
	- Le chunk aura comme propriétés l'item reader, l'item processor et l'item writer définis plus haut.
	- Le paramètre commit-interval du chunk définit le nombre d'items qui vont être stockés en mémoire avant d'être pérennisés. 

	
- Création d'une class BatchPersonne avec une méthode main pour lancer le batch
	
Ce qu'il faut ajouter en plus pour faire marcher le projet :
	- pom.xml : 
		- modifier la version de spring-batch-infrastructure et spring-batch-infrastructure en 2.1.8.RELEASE
		- ajouter une dépendance vers le driver mysql
		
	- batch-context.xml :
		- bean idJdbcTemplate
		- bean dataSource
		- bean transactionManager
		- tx:annotation-driven
		- bean jobLauncher
		- bean jobRepository
		
Lancement :
	run configuration maven > 
		- name : tutoriel-spring-batch
		- base directory : ${workspace_loc:/tutoriel-spring-batch}
		- goals : compile exec:java -Dexec.mainClass="com.rija.dev.BatchPersonne" -Dexec.cleanupDaemonThreads=false
		- profiles : skip tests

