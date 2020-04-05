# Week 6
In week 6 heb ik samen met Sebastiaan en Frank ervoor gezorgd dat de Students en Teachers 'tekenbaar' zijn. Hierbij hebben we als inspiratie wat code van Johan's npc's toegepast. We begonnen eerst met een test Student die gemaakt werd in SimulationController. Als we op start klikken in de Simulatie tab, dan zal de tilemap getekent worden in een nieuw canvas samen met de Student.

![DrawableStudent](Recources/Week6\DrawableStudent.PNG)

Om ervoor te zorgen dat de studenten en teachers uit de agenda-module ingeladen worden, zullen deze eerst uit de SaveFile gelezen moeten worden.

        try {
            this.students = new ArrayList<>(fileIO.getStudents());
            this.teachers = new ArrayList<>(fileIO.getTeachers());
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

Daarnaast staat de Student en Teacher lijst ook in de Draw() en Update().