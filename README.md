# Hotel-management-java
import java.util.*;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;

public class Main {
    static class Room {
        int roomNumber;
        String type;
        double pricePerNight;
        int capacity;
        Room(int roomNumber, String type, double pricePerNight, int capacity) {
            this.roomNumber = roomNumber;
            this.type = type;
            this.pricePerNight = pricePerNight;
            this.capacity = capacity;
        }
        public String toString() {
            return "Room " + roomNumber + " - " + type + " - capacity: " + capacity + " - Rs." + pricePerNight + "/night";
        }
    }

    static class Reservation {
        String reservationId;
        int roomNumber;
        String customerName;
        String customerPhone;
        LocalDate checkIn;
        LocalDate checkOut;
        double totalCost;
        Reservation(int roomNumber, String name, String phone, LocalDate checkIn, LocalDate checkOut, double totalCost) {
            this.reservationId = UUID.randomUUID().toString().substring(0, 6);
            this.roomNumber = roomNumber;
            this.customerName = name;
            this.customerPhone = phone;
            this.checkIn = checkIn;
            this.checkOut = checkOut;
            this.totalCost = totalCost;
        }
        public String toString() {
            return "Reservation " + reservationId + " | Room " + roomNumber + " | " + customerName +
                   " (" + customerPhone + ") | " + checkIn + " -> " + checkOut + " | Rs." + totalCost;
        }
        boolean overlaps(LocalDate start, LocalDate end) {
            return (checkIn.isBefore(end) || checkIn.equals(end)) && (checkOut.isAfter(start) || checkOut.equals(start));
        }
    }

    static class Hotel {
        List<Room> rooms = new ArrayList<>();
        List<Reservation> reservations = new ArrayList<>();
        Hotel() {
            rooms.add(new Room(101, "Single", 1200, 1));
            rooms.add(new Room(102, "Single", 1200, 1));
            rooms.add(new Room(201, "Double", 2200, 2));
            rooms.add(new Room(202, "Double", 2200, 2));
            rooms.add(new Room(301, "Suite", 5000, 4));
        }
        void listRooms() { for (Room r : rooms) System.out.println(r); }
        List<Room> findByType(String type) {
            List<Room> res = new ArrayList<>();
            for (Room r : rooms) if (r.type.equalsIgnoreCase(type)) res.add(r);
            return res;
        }
        boolean available(int roomNum, LocalDate in, LocalDate out) {
            for (Reservation r : reservations) if (r.roomNumber == roomNum && r.overlaps(in, out)) return false;
            return true;
        }
        Room findRoom(int num) { for (Room r : rooms) if (r.roomNumber == num) return r; return null; }
        Reservation book(int num, String name, String phone, LocalDate in, LocalDate out) {
            Room r = findRoom(num);
            if (r == null) throw new IllegalArgumentException("Room not found.");
            if (!available(num, in, out)) throw new IllegalArgumentException("Room not available.");
            long nights = ChronoUnit.DAYS.between(in, out);
            if (nights <= 0) nights = 1;
            double cost = nights * r.pricePerNight;
            Reservation res = new Reservation(num, name, phone, in, out, cost);
            reservations.add(res);
            return res;
        }
        boolean cancel(String id) { return reservations.removeIf(r -> r.reservationId.equalsIgnoreCase(id)); }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        Hotel hotel = new Hotel();
        DateTimeFormatter df = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        boolean run = true;

        System.out.println("=== HOTEL RESERVATION SYSTEM ===");
        while (run) {
            System.out.println("1. View Rooms\n2. Find Rooms by Type\n3. Book Room\n4. Cancel Reservation\n5. View Reservations\n0. Exit");
            System.out.print("Choose: ");
            String ch = sc.nextLine();
            switch (ch) {
                case "1":
                    hotel.listRooms();
                    break;
                case "2":
                    System.out.print("Enter type (Single/Double/Suite): ");
                    String type = sc.nextLine();
                    List<Room> res = hotel.findByType(type);
                    if (res.isEmpty()) System.out.println("No rooms found.");
                    else res.forEach(System.out::println);
                    break;
                case "3":
                    try {
                        System.out.print("Room number: ");
                        int num = Integer.parseInt(sc.nextLine());
                        System.out.print("Name: ");
                        String name = sc.nextLine();
                        System.out.print("Phone: ");
                        String phone = sc.nextLine();
                        System.out.print("Check-in (yyyy-mm-dd): ");
                        LocalDate in = LocalDate.parse(sc.nextLine(), df);
                        System.out.print("Check-out (yyyy-mm-dd): ");
                        LocalDate out = LocalDate.parse(sc.nextLine(), df);
                        Reservation r = hotel.book(num, name, phone, in, out);
                        System.out.println("Booked Successfully!");
                        System.out.println(r);
                    } catch (Exception e) {
                        System.out.println("Error: " + e.getMessage());
                    }
                    break;
                case "4":
                    System.out.print("Enter Reservation ID: ");
                    String id = sc.nextLine();
                    System.out.println(hotel.cancel(id) ? "Canceled." : "ID not found.");
                    break;
                case "5":
                    if (hotel.reservations.isEmpty()) System.out.println("No reservations.");
                    else hotel.reservations.forEach(System.out::println);
                    break;
                case "0":
                    run = false;
                    System.out.println("Goodbye!");
                    break;
                default:
                    System.out.println("Invalid choice.");
            }
            System.out.println();
        }
        sc.close();
    }
}