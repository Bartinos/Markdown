# Week 9


In week 9 heb ik gezeten aan de PathFinding, DistanceMaps voor lokalen en stoelen, en nog een hoop andere dingen. 
Als eerste heb ik, met een beetje hulp van iemand uit een andere klas, de klassen DistanceMap en PathFinding toegevoegd. De DistanceMap bestaat onder andere uit een recalculate() methode. Deze methode voegt voor elke tile die niet onder de collision valt een waarde toe, deze waarde zal de afstand in aantal tiles vanaf de begin tile. De DistanceMap zal dus een begin x en y nodig hebben en de collision layer. Als deze er zijn kan de distanceMap berekent worden.

```
package tileMap;

import java.awt.*;
import java.util.*;

public class DistanceMap {
    private int[][] collisionMap;
    private int targetX;
    private int targetY;
    private boolean[][] map = new boolean[60][32];
    private double[][] distanceMap = new double[60][32];


    public DistanceMap(int[][] collisionMap, int targetX, int targetY) {
        this.collisionMap = collisionMap;
        this.targetX = targetX;
        this.targetY = targetY;

        recalculate();
    }

    public void recalculate() {
        for (int y = 0; y < 32; y++){
            for (int x = 0; x < 60; x++){
                if (collisionMap[y][x] != 0){
                    map[x][y] = true;
                }
                else map[x][y] = false;
            }
        }

        map[targetX][targetY] = false;
        for (int x = 0; x < 60; x++) {
            for (int y = 0; y < 32; y++) {
                distanceMap[x][y] = Integer.MAX_VALUE;
            }
        }

        Queue<Point> points = new LinkedList<>();
        points.offer(new Point(targetX, targetY));
        distanceMap[targetX][targetY] = 0;
        while (!points.isEmpty()) {
            Point p = points.poll();

            for (int x = -1; x <= 1; x++) {
                for (int y = -1; y <= 1; y++) {
                    // Check new point is inside field
                    if (p.x + x < 0 || p.x + x >= 60 || p.y + y < 0 || p.y + y >= 32 || Math.abs(x) == Math.abs(y)) {
                        continue;
                    }
                    double d = distanceMap[p.x][p.y] + Math.sqrt(x * x + y * y);
                    if (d < distanceMap[p.x + x][p.y + y] && !map[p.x + x][p.y + y]) {
                        distanceMap[p.x + x][p.y + y] = d;
                        points.offer(new Point(p.x + x, p.y + y));
                    }
                }
            }
        }
    }


    public double[][] getDistanceMap() {
        return distanceMap;
    }
}
```

Daarnaast heb ik een PathfindLogic klasse gemaakt, deze klasse is onderandere verantwoordelijk voor het geven van de positie aan de Student of Teacher die de methode getPath(Point2D position, String target) aanroept. Eerst kijkt de PathfindLogic op welke tile de student begint, dan wordt er gekeken welke tile de beste optie is. Dan return het de Point2D van de tile. Voordat de PathfindLogic wilt werken, zal eerst de generate() methode gevraagd moeten worden. Deze initialised de DistanceMaps. Het voegt DistanceMaps toe aan elke kamer, daarnaast krijgt elke kamer met stoelen een lijst met de Stoel klasse, die bestaat uit DistanceMap, boolean isTaken en reservedStudent. Nu heeft elke stoel zijn eigen distanceMap, door de isTaken en reservedStudent kan een student zien of een stoel bezet is of niet. Als een stoel bezet is, maar de student is ook de reservedStudent, dan zal hij daarnaartoe blijven lopen.

```package pathFinding;

import data.Classroom;
import pathFinding.Chair;
import pathFinding.DistanceMap;
import tileMap.Target;

import javax.json.*;
import java.awt.geom.Point2D;
import java.util.ArrayList;
import java.util.HashMap;

public class PathfindLogic {
    private Target target;
    private HashMap<String, DistanceMap> distanceHashMaps = new HashMap<>();
    private HashMap<String, Integer> classroomHashMap;
    private ArrayList classroomCodesArrayList;
    private ArrayList<Classroom> classroomArrayList;
    private String fileName;
    private Point2D[][] tileTargets = new Point2D[60][32];

    public PathfindLogic(String fileName) {
        this.fileName = fileName;
        this.target = new Target(fileName);
        this.classroomHashMap = target.getClassroomsHashMap();
        this.classroomCodesArrayList = target.getClassroomCodesList();
        this.classroomArrayList = target.getClassroomsList();
    }

    public void generate() {
        for (String classroom : classroomHashMap.keySet()) {
            int targetX = target.getX(classroomCodesArrayList.indexOf(classroom)) / 32;
            int targetY = target.getY(classroomCodesArrayList.indexOf(classroom)) / 32;
            int location = classroomCodesArrayList.indexOf(classroom);
            distanceHashMaps.put(classroom, new DistanceMap(get2DArrayCollisionLayer(), targetX, targetY));

            for (int y = 0; y < 32; y++) {
                for (int x = 0; x < 60; x++) {
                    tileTargets[x][y] = new Point2D.Double(x * 32 + 16, y * 32 + 16);
                }
            }
            if (target.getWidth(location) == 224) {
                ArrayList<Chair> chairs = new ArrayList<>();
                for (int y = 0; y < 4; y++) {
                    for (int x = 0; x < 3; x++) {
                        if (target.getName(location).equals("101s") || target.getName(location).equals("102s") || target.getName(location).equals("105s") || target.getName(location).equals("106s")) {
                            chairs.add(new Chair(new DistanceMap(get2DArrayCollisionLayer(), targetX + (3 * x), targetY + y), false));
                        } else {
                            chairs.add(new Chair(new DistanceMap(get2DArrayCollisionLayer(), targetX + (3 * x), targetY + y), true));
                        }
                    }
                }
                classroomArrayList.get(location).setChairs(chairs);
                System.out.println(classroomArrayList.get(location) + " Initialized");
            } else if (target.getName(location).equals("Canteen")) {
                ArrayList<Chair> chairs = new ArrayList<>();

                for (int y = 0; y < 8; y++) {
                    for (int xE = 0; xE < 2 ; xE++) {
                        chairs.add(new Chair(new DistanceMap(get2DArrayCollisionLayer(), targetX + (5 * xE), targetY + y), true));
                    }
                    for (int xW = 0; xW < 2; xW++) {
                        chairs.add(new Chair(new DistanceMap(get2DArrayCollisionLayer(), targetX + 4 + (5 * xW), targetY + y), false));
                    }
                }
                for (int y = 0; y < 8; y++) {
                    for (int xE = 0; xE < 2 ; xE++) {
                        chairs.add(new Chair(new DistanceMap(get2DArrayCollisionLayer(), targetX + (5 * xE), targetY + 11 + y), true));
                    }
                    for (int xW = 0; xW < 2; xW++) {
                        chairs.add(new Chair(new DistanceMap(get2DArrayCollisionLayer(), targetX + 4 + (5 * xW), targetY + 11 + y), false));
                    }
                }

                classroomArrayList.get(location).setChairs(chairs);
                System.out.println(classroomArrayList.get(location) + " Initialized");
            }
        }
    }

    public int[][] get2DArrayCollisionLayer() {
        JsonReader reader = Json.createReader(getClass().getResourceAsStream("/Tilemap.json"));
        JsonObject root = reader.readObject();
        for (int i = 0; i < root.getJsonArray("layers").size(); i++) {
            if (root.getJsonArray("layers").getJsonObject(i).getString("name").equals("Collision")) {
                JsonArray collisionArray = root.getJsonArray("layers").getJsonObject(i).getJsonArray("data");
                int[][] collision2DArray = new int[32][60];
                int j = 0;
                for (int y = 0; y < 32; y++) {
                    for (int x = 0; x < 60; x++) {
                        collision2DArray[y][x] = collisionArray.getInt(j);
                        j++;
                    }
                }
                return collision2DArray;
            }
        }
        System.out.println("Failed to get Collision layer");
        return null;
    }

    public DistanceMap getDistanceMap() {
        return classroomArrayList.get(5).getChairs().get(4).getDistanceMap();
    }

    public Point2D getPath(Point2D position, String target) {
        DistanceMap targetField = distanceHashMaps.get(target);
        return calculatePath(position, targetField);
    }

    public Point2D getPath(Point2D position, DistanceMap distanceMap) {
        DistanceMap targetField = distanceMap;
        return calculatePath(position, targetField);
    }

    private Point2D calculatePath(Point2D position, DistanceMap targetField) {
        int currentTileX = (int) Math.floor((position.getX()) / 32);
        int currentTileY = (int) Math.floor((position.getY()) / 32);
        double currentDistance = targetField.getDistanceMap()[currentTileX][currentTileY];
        if (currentDistance > targetField.getDistanceMap()[currentTileX + 1][currentTileY] && targetField.getDistanceMap()[currentTileX + 1][currentTileY] != Integer.MAX_VALUE) {
            return tileTargets[currentTileX + 1][currentTileY];
        }

        if (currentDistance > targetField.getDistanceMap()[currentTileX][currentTileY + 1] && targetField.getDistanceMap()[currentTileX][currentTileY + 1] != Integer.MAX_VALUE) {
            return tileTargets[currentTileX][currentTileY + 1];
        }
        if (currentDistance > targetField.getDistanceMap()[currentTileX - 1][currentTileY] && targetField.getDistanceMap()[currentTileX - 1][currentTileY] != Integer.MAX_VALUE) {
            return tileTargets[currentTileX - 1][currentTileY];
        }
        if (currentDistance > targetField.getDistanceMap()[currentTileX][currentTileY - 1] && targetField.getDistanceMap()[currentTileX][currentTileY - 1] != Integer.MAX_VALUE) {
            return tileTargets[currentTileX][currentTileY - 1];
        }
        return tileTargets[currentTileX][currentTileY - 1];
    }

    public ArrayList<Classroom> getClassroomArrayList() {
        return classroomArrayList;
    }
}
```

Verder heb ik 2 edit pop-ups toegevoegd aan PersonManagerm, omdat ik niet tevreden was met hoe de originele edit werkten. Als er een student geselecteerd wordt en er wordt op edit gedrukt, dan zal er een edit pop-up verschijnen. Deze pop-up staat als methode in de GuiController en belemmerd alle interactie met de hoofd window. In de pop-up kan je de eigenschappen van de student of teacher veranderen.

![Teacher edit pop-up](Recources/Week9/EditPop-up.png)

Daarnaast heb ik er dus voor gezorgd dat Students naar eigen plaatsen gaan in de les waar ze naar toe gaan. Elk lokaal heeft 12 stoelen, de kantine heeft er 64. Omdat er 72 studenten in totaal aanwezig kunnen zijn, mogen de student in de kantine 'in' elkaar zitten. Als ze naar een lokaal gaan is dat niet zo. Aan het begin van de dag zullen leerlingen naar de kantine gaan, ze genereren dan hun eigen chairIndex, van de 64 stoelen zullen ze dus een random stoel pakken. Als de studenten naar een les gaan, dan zal er een boolean geswitched worden. Op trigger van deze switch, zal de student een random getal tussen de 0 en 12 genereren en gaan naar die stoel. Is de stoel waar ze naartoe willen al bezet, dan zullen ze opnieuw een random waarde genereren. Als de les afgelopen is, dan zal de stoel waar de student zat weer vrij zijn. Daarnaast wordt de boolean weer geswichted, omdat deze nu negatief is, zal er een random getal tussen de 0 en 64 gegenereerd worden, dat is dus de stoel in de kantine. Verder ben ik onderandere verantwoordelijk voor dat de studenten en teachers de juiste lessen volgen en naar de juiste plekken gaan.
```
        if (this.lessons.size() > currentLesson){
            Lesson lesson = lessons.get((int) currentLesson);
            int[] beginTime = lesson.getBeginLesson();
            int[] endTime = lesson.getEndLesson();
            String locationS = lesson.getClassroom().getClassNumber() + "s";

            ArrayList<Chair> classroomChairsList = classroomList.get(classroomCodesList.indexOf(locationS)).getChairs();
            ArrayList<Chair> canteenChairsList = classroomList.get(classroomCodesList.indexOf("canteen")).getChairs();

            lastRoom = classroomList.get(classroomCodesList.indexOf(locationS));

            if (hour >= beginTime[0] && minute >= beginTime[1] && hour <= endTime[0] && minute <= endTime[1] && this.group.getCode().equals(lesson.getGroup().getCode())) {
                canteenChairsList.get(chairIndex).setTaken(false, null);
                searchForChair.set(true);
                while (classroomChairsList.get(chairIndex).isTaken() && this != classroomChairsList.get(chairIndex).getReservedStudent())
                    chairIndex = new Random().nextInt(12);
                classroomChairsList.get(chairIndex).setTaken(true, this);
                this.setTarget(pathfindLogic.getPath(this.getPosition(), classroomChairsList.get(chairIndex).getDistanceMap()));

            } else {
                if (chairIndex < 12)
                classroomChairsList.get(chairIndex).setTaken(false, null);
                searchForChair.set(false);
                if (this.lessons.size() > currentLesson) {
                    this.setTarget(pathfindLogic.getPath(this.getPosition(), canteenChairsList.get(chairIndex).getDistanceMap()));
                    canteenChairsList.get(chairIndex).setTaken(true, this);
                }
            }
        } else {
            ArrayList<Chair> canteenChairsList = classroomList.get(classroomCodesList.indexOf("canteen")).getChairs();
            if (chairIndex > 12) {
                searchForChair.set(true);
                searchForChair.set(false);
            }
            this.setTarget(pathfindLogic.getPath(this.getPosition(), canteenChairsList.get(chairIndex).getDistanceMap()));
            canteenChairsList.get(chairIndex).setTaken(true, this);
        }
    }

    private void setRandomChairIndex(ObservableValue<? extends Boolean> observable, Boolean oldValue, Boolean newValue){
        if (newValue)
            chairIndex = new Random().nextInt(12);
        else
            chairIndex = new Random().nextInt(64);

        System.out.println("Before: " + (int) currentLesson);
        currentLesson += 0.5;
        System.out.println("After: " + (int) currentLesson);
    }
```
