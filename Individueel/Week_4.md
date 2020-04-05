# Week 4

In week 4 zijn David en ik verder gegaan met de agenda-module. Ik heb ervoor gezorgd dat Roster nu ook de beschikbare groepen toont. 

        teachersTable.getItems().addListener((ListChangeListener<Teacher>) c -> {
            comboTeacherNameList = teachersTable.getItems();
            System.out.println("Teacher added to combobox");
            teacherAllNameBox.setItems(comboTeacherNameList);
        });
        studentsTable.getItems().addListener((ListChangeListener<Student>) c -> {
            groupSet = new HashSet<>();
            for (Student student: studentsTable.getItems()) {
                groupSet.add(student.getStudentGroup());
            }
            comboStudentGroupRoster.clear();
            comboStudentGroupRoster.addAll(groupSet);
            studentGroupBoxs.setItems(comboStudentGroupRoster);
**PersonManager**

![GroupsetPM](Recources/Week4/GroupsetPM.png)

**Roster**

![GroupsetPM](Recources/Week4/GroupsetRoster.png)

Verder heb ik de Lesson Addbutton werkend gemaakt. De les bestaat nu ook uit een Teacher, Group, Classroom, begintijd en eintijd. Daarnaast heb ik geexperimenteerd met FileIO om te kijken of ik het kan implementeren in het project.

# Week 4.5: vakantie

Deze week heb ik de FileIO klasse gemaakt na ik er veel geoefent mee heb. De FileIO is uiteraard verantwoordelijk van het opslaan van de gegevens en het inladen ervan. De gegevens die opgeslagen worden zijn 3 observableArrayLists; alle studenten, leraren en lessen. Om dit werkent te laten maken, moesten alle klassen die betrokken waren tot Student, Teacher en Lesson, Serializable worden. Daarnaast was niet alles te schrijven in sommige van deze klassen, SimpleStringPorperty in Student is niet Serializable. Om ervoor tezorgen dat deze niet problemen veroorzaken tijdens het opslaan, zijn ze transient. Daarnaast heb ik ook costum writers en readers gemaakt voor deze 3 klassen.

**FileIO klasse**

```package gui;

import data.Student;
import data.Teacher;
import data.Lesson;
import javafx.collections.FXCollections;
import javafx.collections.ObservableList;

import java.io.*;
import java.util.ArrayList;
import java.util.Scanner;

import static javafx.collections.FXCollections.observableArrayList;


@SuppressWarnings("Duplicates")
public class FileIO {
    private Scanner scanner;
    private File file;
    private FileWriter fileWriter;

    public FileIO() {

    }

    public void writeAll(ObservableList<Student> obStudents, ObservableList<Teacher> obTeachers, ObservableList<Lesson> obLessons) throws IOException, ClassNotFoundException {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("src/SaveFile"));

        ArrayList<Student> allStudents = new ArrayList<>(obStudents);
        ArrayList<Teacher> allTeachers = new ArrayList<>(obTeachers);
        ArrayList<Lesson> allLessons = new ArrayList<>(obLessons);

        oos.writeObject(allStudents);
        oos.writeObject(allTeachers);
        oos.writeObject(allLessons);

        oos.close();
    }

    public ObservableList getStudents() throws IOException, ClassNotFoundException {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("src/SaveFile"));
        ArrayList<Student> loadedStudents = (ArrayList<Student>) ois.readObject();
        ObservableList<Student> ObsStudents = observableArrayList(loadedStudents);
        ois.close();
        return ObsStudents;

    }

    public ObservableList getTeachers() throws IOException, ClassNotFoundException {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("src/SaveFile"));
        ois.readObject();
        ArrayList<Teacher> loadedTeachers = (ArrayList<Teacher>) ois.readObject();
        ObservableList<Teacher> ObsTeachers = observableArrayList(loadedTeachers);
        ois.close();
        return ObsTeachers;
    }

    public ObservableList getLessons() throws IOException, ClassNotFoundException {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("src/SaveFile"));
        ois.readObject();
        ois.readObject();
        ArrayList<Lesson> loadedLessons = (ArrayList<Lesson>) ois.readObject();
        ObservableList<Lesson> ObsLessons = observableArrayList(loadedLessons);
        ois.close();
        return ObsLessons;
    }
}
```

**Voorbeeld van een costum writer en reader (Student in dit geval)**

    private void writeObject(ObjectOutputStream oos) throws IOException {
        oos.defaultWriteObject();
        oos.writeObject(groupTBST.get());
        oos.writeUTF(genderTBST.get());
        oos.writeUTF(lastNameTBST.get());
    }
    private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {
        ois.defaultReadObject();
        groupTBST = new SimpleObjectProperty(ois.readObject());
        genderTBST = new SimpleStringProperty(ois.readUTF());
        lastNameTBST = new SimpleStringProperty(ois.readUTF());
    }

In de main, primaryStage, staat er een setOnCloseRequest met daarin gevraagd naar de save methode die ik daar neer heb gezet. Hierdoor slaat de agenda module al de gegevens op als het afgesloten wordt. Deze gegevens worden allemaal in de SaveFile opgeslagen die in de gitIgnore staat. Verder heb ik het Plan van Aanpak nog gecontroleerd die Jorn verbeterd heeft en er feedback op gegeven.