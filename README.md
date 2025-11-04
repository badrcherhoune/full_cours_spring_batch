## **1. Introduction à Spring Batch**

Spring Batch est un framework **Java** conçu pour le traitement par lots (**batch processing**). Il est utilisé pour exécuter des opérations répétitives sur de gros volumes de données, avec des fonctionnalités telles que :

* Lecture de données depuis différentes sources (fichiers, bases de données, API…)
* Traitement des données (transformation, validation)
* Écriture des résultats (dans fichiers, bases de données…)
* Gestion des transactions, redémarrage, suivi d’exécution (**job repository**)
* Gestion des erreurs et logs

Un **batch job** est composé de **jobs**, eux-mêmes constitués de **steps**.

---

## **2. Concepts principaux du diagramme**

### **2.1 JobLauncher**

* **Rôle** : Point d’entrée pour démarrer un job batch.
* **Méthode principale** : `run(Job job, JobParameters params)`
  Elle démarre un job avec des paramètres.
* **Exemple d’implémentation** : `SimpleJobLauncher` (une implémentation de l’interface `JobLauncher`)

---

### **2.2 Job et Step**

* **Job**

  * Représente l’exécution d’un batch complet.
  * Composé d’un ou plusieurs **steps**.
  * Créé par `JobBuilderFactory`.

* **Step**

  * Représente une unité de travail dans un job.
  * Contient les trois composants clés :

    1. **ItemReader** : lit les données
    2. **ItemProcessor** : traite ou transforme les données
    3. **ItemWriter** : écrit les données transformées
  * Créé par `StepBuilderFactory`.

---

### **2.3 ItemReader / ItemProcessor / ItemWriter**

* **ItemReader** : lecture séquentielle des données (ex : lecture depuis CSV, DB, API…)
* **ItemProcessor** : transformation ou validation de la donnée
* **ItemWriter** : écriture finale dans la destination (ex : DB, fichier…)

C’est le **pattern chunk-oriented processing** : lire → traiter → écrire.

---

### **2.4 JobParameters et JobParametersBuilder**

* **JobParameters** : paramètres passés au job (ex : date du jour, identifiant, fichier à traiter…)
* **JobParametersBuilder** : permet de construire ces paramètres facilement
* Exemple : `new JobParametersBuilder().addString("fileName", "data.csv").toJobParameters();`

---

### **2.5 JobBuilderFactory et StepBuilderFactory**

* Classes utilitaires pour créer des jobs et des steps facilement
* **JobBuilderFactory** → crée des jobs
* **StepBuilderFactory** → crée des steps
* Ils utilisent le pattern **builder** pour la configuration fluide.

---

### **2.6 JobExecutionListener**

* Interface pour écouter les événements d’un job
* Méthodes clés :

  * `beforeJob(JobExecution jobExecution)` → avant le démarrage
  * `afterJob(JobExecution jobExecution)` → après la fin du job
* Utilisé pour le logging ou des actions avant/après le job

---

## **3. Flow d’exécution d’un job**

1. **Création des paramètres du job**

   * Utilisation de `JobParametersBuilder` pour générer `JobParameters`.
2. **Création du job**

   * Avec `JobBuilderFactory`, on définit les steps.
3. **Création des steps**

   * Avec `StepBuilderFactory` : on configure ItemReader, ItemProcessor et ItemWriter.
4. **Exécution**

   * `JobLauncher.run(job, jobParameters)` démarre le job.
5. **Listener**

   * `JobExecutionListener` peut exécuter du code avant ou après le job.

<img src="Ch02_SpringBatchArchitecture_Overview_MainComponents.png" alt="Diagram"/>
---

## **4. Exemple de configuration Spring Batch**

```java
@Configuration
@EnableBatchProcessing
public class BatchConfig {

    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    @Bean
    public ItemReader<String> reader() {
        return new FlatFileItemReaderBuilder<String>()
                   .name("reader")
                   .resource(new FileSystemResource("data.csv"))
                   .lineMapper(new DefaultLineMapper<>())
                   .build();
    }

    @Bean
    public ItemProcessor<String, String> processor() {
        return item -> item.toUpperCase(); // exemple simple
    }

    @Bean
    public ItemWriter<String> writer() {
        return items -> items.forEach(System.out::println);
    }

    @Bean
    public Step step() {
        return stepBuilderFactory.get("step1")
                .<String, String>chunk(10)
                .reader(reader())
                .processor(processor())
                .writer(writer())
                .build();
    }

    @Bean
    public Job job(JobExecutionListener listener) {
        return jobBuilderFactory.get("job1")
                .listener(listener)
                .start(step())
                .build();
    }
}
```

* `chunk(10)` → traite 10 éléments à la fois avant d’écrire
* Listener est utilisé pour écouter la fin ou le début du job

---

## **5. Points clés à retenir**

* **JobLauncher** : lance les jobs
* **Job** : ensemble de steps
* **Step** : unité de travail avec reader → processor → writer
* **JobParameters** : paramètres dynamiques pour le job
* **Listeners** : actions avant/après job
* **Factories** (`JobBuilderFactory` / `StepBuilderFactory`) : créent facilement jobs et steps
* Spring Batch gère **transactions, redémarrage, erreurs et logging** automatiquement.

