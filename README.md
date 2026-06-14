# Attendance---System
This is based on java

import java.io.*;
import java.util.*;

class Student {
    int id;
    String rollNo;   // ✔ String (fixed)
    String name;
    List<Integer> attendance;

    Student(int id, String rollNo, String name, List<Integer> attendance) {
        this.id = id;
        this.rollNo = rollNo;
        this.name = name;
        this.attendance = attendance;
    }

    int getTotalPresent() {
        int sum = 0;
        for (int a : attendance) sum += a;
        return sum;
    }

    double getPercentage(int totalLectures) {
        return (getTotalPresent() * 100.0) / totalLectures;
    }
}

public class AttendanceSystem {

    static List<Student> students = new ArrayList<>();
    static List<String> dates = new ArrayList<>();

    // Load CSV
    static void loadData(String fileName) throws Exception {
        BufferedReader br = new BufferedReader(new FileReader(fileName));
        String line = br.readLine();

        String[] header = line.split(",");
        for (int i = 3; i < header.length; i++) {
            dates.add(header[i]);
        }

        int idCounter = 1;

        while ((line = br.readLine()) != null) {
            String[] data = line.split(",");

            String roll = data[1];   // ✔ FIXED
            String name = data[2];

            List<Integer> att = new ArrayList<>();
            for (int i = 3; i < data.length; i++) {
                att.add(Integer.parseInt(data[i]));
            }

            students.add(new Student(idCounter++, roll, name, att));
        }

        br.close();
    }

    static int totalLectures() {
        return dates.size();
    }

    // Q1 Class Summary
    static void classSummary() {
        int totalStudents = students.size();
        int totalLectures = totalLectures();

        double avg = 0;
        for (Student s : students) {
            avg += s.getPercentage(totalLectures);
        }
        avg /= totalStudents;

        int max = -1, min = Integer.MAX_VALUE;
        String maxDate = "", minDate = "";

        for (int i = 0; i < dates.size(); i++) {
            int sum = 0;
            for (Student s : students) {
                sum += s.attendance.get(i);
            }

            if (sum > max) {
                max = sum;
                maxDate = dates.get(i);
            }
            if (sum < min) {
                min = sum;
                minDate = dates.get(i);
            }
        }

        System.out.println("\n--- Class Summary ---");
        System.out.println("Total Students: " + totalStudents);
        System.out.println("Total Lectures: " + totalLectures);
        System.out.println("Average Attendance: " + avg);
        System.out.println("Max Attendance Date: " + maxDate);
        System.out.println("Min Attendance Date: " + minDate);
    }

    // Q2 List Students
    static void listStudents() {
        System.out.println("\n--- Student List ---");

        students.sort(Comparator.comparing(s -> s.rollNo));

        for (Student s : students) {
            System.out.println(s.rollNo + " " + s.name + " " +
                    String.format("%.2f", s.getPercentage(totalLectures())) + "%");
        }
    }

    // Q3 Top N
    static void topN(int n) {
        System.out.println("\n--- Top " + n + " Students ---");

        students.stream()
                .sorted((a, b) -> Double.compare(
                        b.getPercentage(totalLectures()),
                        a.getPercentage(totalLectures())))
                .limit(n)
                .forEach(s -> System.out.println(s.name));
    }

    // Q4 Bottom N
    static void bottomN(int n) {
        System.out.println("\n--- Bottom " + n + " Students ---");

        students.stream()
                .sorted(Comparator.comparingDouble(
                        s -> s.getPercentage(totalLectures())))
                .limit(n)
                .forEach(s -> System.out.println(s.name));
    }

    // Q5 Defaulters
    static void defaulters(double threshold) {
        System.out.println("\n--- Defaulters (< " + threshold + "%) ---");

        int totalLectures = totalLectures();
        int count = 0;

        for (Student s : students) {
            double percent = s.getPercentage(totalLectures);

            if (percent < threshold) {
                count++;

                int attended = s.getTotalPresent();
                int required = (int)Math.ceil((threshold * totalLectures) / 100.0);
                int missing = required - attended;

                System.out.println(s.name +
                        " | Attended: " + attended +
                        " | Need More: " + missing);
            }
        }

        System.out.println("Total Defaulters: " + count);
    }

    // Q6 Categorization
    static void categorize() {
        int c1=0,c2=0,c3=0,c4=0;

        for (Student s : students) {
            double p = s.getPercentage(totalLectures());

            if (p >= 90) c1++;
            else if (p >= 75) c2++;
            else if (p >= 50) c3++;
            else c4++;
        }

        System.out.println("\n--- Categories ---");
        System.out.println(">=90%: " + c1);
        System.out.println("75-90%: " + c2);
        System.out.println("50-75%: " + c3);
        System.out.println("<50%: " + c4);
    }

    // Q7 Date-wise Analysis
    static void dateWise() {
        System.out.println("\n--- Date-wise Analysis ---");

        for (int i = 0; i < dates.size(); i++) {
            int sum = 0;

            for (Student s : students) {
                sum += s.attendance.get(i);
            }

            System.out.println(dates.get(i) +
                    " | Lectures: 1 | Total Attendance: " + sum);
        }
    }

    // Q8 Report Generation
    static void generateReport(String fileName) throws Exception {
        PrintWriter pw = new PrintWriter(new FileWriter(fileName));

        pw.println("Attendance Report");
        pw.println("Total Students: " + students.size());
        pw.println("Total Lectures: " + totalLectures());

        pw.println("\nStudent List:");
        for (Student s : students) {
            pw.println(s.rollNo + " " + s.name + " " +
                    String.format("%.2f", s.getPercentage(totalLectures())) + "%");
        }

        pw.close();
        System.out.println("\nReport Generated Successfully!");
    }

    // MAIN METHOD
    public static void main(String[] args) throws Exception {

        Scanner sc = new Scanner(System.in);

        loadData("SubjectWiseAttendance.CSV");

        classSummary();
        listStudents();

        System.out.print("\nEnter N for Top Students: ");
        int n = sc.nextInt();
        topN(n);

        System.out.print("Enter N for Bottom Students: ");
        n = sc.nextInt();
        bottomN(n);

        System.out.print("Enter Attendance Threshold: ");
        double t = sc.nextDouble();
        defaulters(t);

        categorize();
        dateWise();

        generateReport("report.txt");
    }
}
