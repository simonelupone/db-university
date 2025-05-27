# db-university

Modellizzare la struttura di un database per memorizzare tutti i dati riguardanti una università:

sono presenti diversi Dipartimenti (es.: Lettere e Filosofia, Matematica, Ingegneria ecc.);

- ogni Dipartimento offre più Corsi di Laurea (es.: Civiltà e Letterature Classiche, Informatica, Ingegneria Elettronica
  ecc..)
- ogni Corso di Laurea prevede diversi Corsi (es.: Letteratura Latina, Sistemi Operativi 1, Analisi Matematica 2 ecc.);
- ogni Corso può essere tenuto da diversi Insegnanti;
- ogni Corso prevede più appelli d'Esame;
- ogni Studente è iscritto ad un solo Corso di Laurea;
- ogni Studente può iscriversi a più appelli di Esame;
- per ogni appello d'Esame a cui lo Studente ha partecipato, è necessario memorizzare il voto ottenuto, anche se non
  sufficiente.

Pensiamo a quali entità (tabelle) creare per il nostro database e cerchiamo poi di stabilirne le relazioni. Infine,
andiamo a definire le colonne e i tipi di dato di ogni tabella.
Utilizzare https://www.drawio.com/ per la creazione dello schema.

# Esercizio

1. Selezionare tutti gli studenti nati nel 1990 (160)
   ```
   SELECT *
   FROM `students`
   WHERE YEAR(`date_of_birth`) = 1990;
   ```
2. Selezionare tutti i corsi che valgono più di 10 crediti (479)
   ```
   SELECT *
   FROM `courses`
   WHERE `cfu` > 10;
   ```
3. Selezionare tutti gli studenti che hanno più di 30 anni

    ```
    SELECT *, TIMESTAMPDIFF(YEAR, `date_of_birth`, CURDATE()) AS `age`
    FROM `students`
    WHERE TIMESTAMPDIFF(YEAR, `date_of_birth`, CURDATE()) > 30
    ORDER BY `age` ASC;
    ```
4. Selezionare tutti i corsi del primo semestre del primo anno di un qualsiasi corso di
   laurea (286)

    ```
    SELECT *
    FROM `courses`
    WHERE `period` = 'I semestre' AND `year` = 1;
    ```
5. Selezionare tutti gli appelli d'esame che avvengono nel pomeriggio (dopo le 14) del
   20/06/2020 (21)

    ```
    SELECT *
    FROM `exams`
    WHERE HOUR(`hour`) >= 14 AND `date` = '2020-06-20';
    ```
6. Selezionare tutti i corsi di laurea magistrale (38)
    ```
    SELECT *
    FROM `degrees`
    WHERE `level` = 'magistrale';
    ```
7. Da quanti dipartimenti è composta l'università? (12)

    ```
    SELECT COUNT(*)
    FROM `departments`;
    ```
8. Quanti sono gli insegnanti che non hanno un numero di telefono? (50)

   ```
   SELECT COUNT(*)
   FROM `teachers`
   WHERE `phone` IS NULL;
   ```

---

# Esercizio - JOIN

1. Selezionare tutti gli studenti iscritti al Corso di Laurea in Economia
  ```
    SELECT `students`.`name`, `students`.`registration_number`, `degrees`.`name`
    FROM `students`
    JOIN `degrees` ON `students`.`degree_id` = `degrees`.`id`
    WHERE `degrees`.`name` = 'Corso di Laurea in Economia';
  ```
2. Selezionare tutti i Corsi di Laurea Magistrale del Dipartimento di
Neuroscienze
  ```
  SELECT `degrees`.`name` AS `degree_name`, `departments`.`name` AS `department_name`
  FROM `degrees`
  INNER JOIN `departments`
  ON `degrees`.`department_id` = `departments`.`id`
  WHERE `degrees`.`level` = 'magistrale' AND `departments`.`name` = "Dipartimento di Neuroscienze";
  ```

3. Selezionare tutti i corsi in cui insegna Fulvio Amato (id=44)
  ```
  SELECT `courses`.`name` AS `course_name`, `courses`.`year`, `courses`.`period`, `teachers`.`name` AS `teacher_name`, `teachers`.`surname` AS `teacher_surname`
  FROM `courses`
  JOIN `course_teacher` ON `courses`.`id` = `course_teacher`.`course_id`
  JOIN `teachers` ON `course_teacher`.`teacher_id` = `teachers`.`id`
  WHERE `teachers`.`id` = 44;
  ```

4. Selezionare tutti gli studenti con i dati relativi al corso di laurea a cui
sono iscritti e il relativo dipartimento, in ordine alfabetico per cognome e
nome
  ```
  SELECT `students`.`name`, `students`.`surname`, `degrees`.`name` AS `degree_name`, `departments`.`name` AS `department_name`
  FROM `students`
  JOIN `degrees` ON `students`.`degree_id` = `degrees`.`id`
  JOIN `departments` ON `degrees`.`department_id` = `departments`.`id`
  ORDER BY `students`.`surname`, `students`.`name`;
  ```

5. Selezionare tutti i corsi di laurea con i relativi corsi e insegnanti
  ```
  SELECT `degrees`.`name` AS `degree_name`, `courses`.`name` AS `course_name`, `teachers`.`name` AS `teacher_name`, `teachers`.`surname` AS `teacher_surname`
  FROM `courses`
  JOIN `course_teacher` ON `course_teacher`.`course_id` = `courses`.`id`
  JOIN `teachers` ON `course_teacher`.`teacher_id` = `teachers`.`id`
  JOIN `degrees` ON `courses`.`degree_id` = `degrees`.`id`
  ORDER BY `degrees`.`name`;
  ```

6. Selezionare tutti i docenti che insegnano nel Dipartimento di
Matematica (54)
  ```
  SELECT DISTINCT `teachers`.`id`, `teachers`.`name`, `teachers`.`surname`, `departments`.`name`
  FROM `teachers`
  JOIN `course_teacher` ON `course_teacher`.`teacher_id` = `teachers`.`id`
  JOIN `courses` ON `course_teacher`.`course_id` = `courses`.`id`
  JOIN `degrees` ON `courses`.`degree_id` = `degrees`.`id`
  JOIN `departments` ON `degrees`.`department_id` = `departments`.`id`
  WHERE `departments`.`name` = "Dipartimento di Matematica"
  ORDER BY `teachers`.`id`;
  ```

7. BONUS: Selezionare per ogni studente il numero di tentativi sostenuti
per ogni esame, stampando anche il voto massimo. Successivamente,
filtrare i tentativi con voto minimo 18.
  ```
  SELECT COUNT(*), `students`.`registration_number`, `students`.`name`, `students`.`surname`, `exams`.`id` AS `id_exam`, `exams`.`date`, MAX(`exam_student`.`vote`) AS `max_vote`
  FROM `exam_student`
  JOIN `students` ON `students`.`id` = `exam_student`.`student_id`
  JOIN `exams` ON `exams`.`id` = `exam_student`.`exam_id`
  WHERE `exam_student`.`vote` >= 18
  GROUP BY `students`.`registration_number`, `exams`.`id`, `students`.`name`, `students`.`surname`
```

# Esercizio - GROUP BY

1. Contare quanti iscritti ci sono stati ogni anno
  ```
  SELECT YEAR(`enrolment_date`) AS `year`, COUNT(*) AS `students`
  FROM `students`
  GROUP BY `year`;
  ```
2. Contare gli insegnanti che hanno l'ufficio nello stesso edificio
  ```
  SELECT COUNT(*) AS `teachers_number`, `office_address`
  FROM `teachers`
  GROUP BY `office_address`;
  ```
3. Calcolare la media dei voti di ogni appello d'esame
  ```
  SELECT AVG(`vote`) AS `average_vote`, `exam_id`
  FROM `exam_student`
  GROUP BY `exam_id`;
  ```
4. Contare quanti corsi di laurea ci sono per ogni dipartimento
  ```
  SELECT COUNT(*) AS `degrees_number`,  `department_id`
  FROM `degrees`
  GROUP BY `department_id`;
  ```