# Week 7


Hier hebben ik en Sebastiaan pair programmed via discord, omdat we thuis moesten blijven in verband met de corona crisis. We hadden als doel het creëeren van de Target classe en deze toepassen op de Students. Deze klasse is verantwoordelijk voor het uitlezen van de object layer, deze object bevat de informatie van de lokalen en ruimtes waar de studenten en of teachers naar toe moeten op een bepaald tijdstip. Het maken van deze klasse is ons gelukt, we hebben een Timer toegevoegd aan SimulationController met hulp van Frank .In de SimulationController worden nu de students geupdate afhankelijk van les. Dit is nog test code uiteraard. Daarnaast heb ik nog wat bugs gefixt.

                int time[] = lesson.getBeginLesson();

                String locationS = lesson.getClassRoom().getClassNumber() + "s";
                System.out.println(locationS);

                if (hour == time[0] && minute == time[1]){
                    student.setTarget(target.getCenter(classrooms.indexOf(locationS)));
                }
            }
        }

