# ARS
Airline reservation system 
package bookingserviceinterface;

public interface BookingServiceInterface {
    Ticket bookFlight(Passenger p, Flight f);
    boolean cancelTicket(String ticketId);
    void viewAllBookings();
}



public class Person {
    private String name, email, phone;

    public Person(String name, String email, String phone) {
        this.name = name;
        this.email = email;
        this.phone = phone;
    }

    public String getName() { return name; }
    public String getEmail() { return email; }
    public String getPhone() { return phone; }
}


public class Passenger extends Person {
    private String passportNumber;

    public Passenger(String name, String email, String phone, String passportNumber) {
        super(name, email, phone);
        this.passportNumber = passportNumber;
    }

    public String getPassportNumber() { return passportNumber; }
}

public class Flight {
    private String flightNumber, origin, destination;
    private int capacity, bookedSeats;

    public Flight(String flightNumber, String origin, String destination, int capacity) {
        this.flightNumber = flightNumber;
        this.origin = origin;
        this.destination = destination;
        this.capacity = capacity;
        this.bookedSeats = 0;
    }

    public String getFlightNumber() { return flightNumber; }
    public String getOrigin() { return origin; }
    public String getDestination() { return destination; }

    public boolean isAvailable() { return bookedSeats < capacity; }

    public boolean bookSeat() {
        if (isAvailable()) {
            bookedSeats++;
            return true;
        }
        return false;
    }

    public void cancelSeat() {
        if (bookedSeats > 0) bookedSeats--;
    }
}



public class Ticket {
    private String ticketId;
    private Passenger passenger;
    private Flight flight;

    public Ticket(String ticketId, Passenger passenger, Flight flight) {
        this.ticketId = ticketId;
        this.passenger = passenger;
        this.flight = flight;
    }

    public String getTicketId() { return ticketId; }
    
    public Flight getFlight() {
    return flight;
}

    public String toString() {
        return "Ticket ID: " + ticketId + " | Passenger: " + passenger.getName() +
               " | Flight: " + flight.getFlightNumber() + " (" +
               flight.getOrigin() + " -> " + flight.getDestination() + ")";
    }
}
import java.io.*;
import java.util.*;
public class BookingService implements BookingServiceInterface {
    private List<Flight> flights = new ArrayList<>();
    private List<Ticket> tickets = new ArrayList<>();
    private File bookingFile = new File("bookings.txt");
    private int ticketCounter = 1000;
    public BookingService() {
        flights.add(new Flight("AI101", "Delhi", "Mumbai", 2));
        flights.add(new Flight("AI102", "Delhi", "Bangalore", 3));
           // Display the path of the file
   System.out.println("File path: " + bookingFile.getAbsolutePath());
        loadBookingsFromFile();
    }
    public Flight findFlight(String flightNumber) {
        for (Flight f : flights) {
            if (f.getFlightNumber().equalsIgnoreCase(flightNumber)) {
                return f;
            }
        } return null;
    }
public Ticket bookFlight(Passenger p, Flight f) {
        if (f != null && f.bookSeat()) {
            String ticketId = "TKT" + ticketCounter++;
            Ticket ticket = new Ticket(ticketId, p, f);
            tickets.add(ticket);
            saveBookingToFile(ticket);
            return ticket;
        }
        return null;
    }
    public boolean cancelTicket(String ticketId) {
        Iterator<Ticket> iterator = tickets.iterator();
        while (iterator.hasNext()) {
            Ticket t = iterator.next();
            if (t.getTicketId().equalsIgnoreCase(ticketId)) {
                t.getFlight().cancelSeat();
                iterator.remove();
                rewriteFile();
                return true;
            }
        }
        return false;
    }
 public void viewAllBookings() {
        if (tickets.isEmpty()) {
            System.out.println("No bookings found.");
        } else {
            for (Ticket t : tickets) {
                System.out.println(t);
            }
        }
    }
   private void saveBookingToFile(Ticket ticket) {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter("bookings.txt", true))) {
            writer.write(ticket.toString());
            writer.newLine();
        } catch (IOException e) {
            System.out.println("Error saving booking: " + e.getMessage());
        }
    }

    private void loadBookingsFromFile() {
        File file = new File("bookings.txt");
        if (!file.exists()) return;
        try (BufferedReader reader = new BufferedReader(new FileReader(file))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line); // Load display only
            }
        } catch (IOException e) {
            System.out.println("Error loading bookings: " + e.getMessage());
        }
    }
 private void rewriteFile() {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter("bookings.txt"))) {
            for (Ticket t : tickets) {
                writer.write(t.toString());
                writer.newLine();
            }
        } catch (IOException e) {
            System.out.println("Error rewriting bookings: " + e.getMessage());
        }
    }
}
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class AirlineReservationSystem {
    public static void main(String[] args) {
        BookingService bookingService = new BookingService();
        JFrame frame = new JFrame("Airline Reservation System");
        frame.setSize(400, 300);
        frame.setLayout(new FlowLayout());
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JButton bookButton = new JButton("Book Flight");
        JButton cancelButton = new JButton("Cancel Ticket");
        JButton viewButton = new JButton("View Bookings");
 bookButton.addActionListener(e -> {
            String name = JOptionPane.showInputDialog("Enter Name:");
            String email = JOptionPane.showInputDialog("Enter Email:");
            String phone = JOptionPane.showInputDialog("Enter Phone:");
            String passport = JOptionPane.showInputDialog("Enter Passport Number:");
            String flightNum = JOptionPane.showInputDialog("Enter Flight Number:");
Passenger passenger = new Passenger(name, email, phone, passport);
            Flight flight = bookingService.findFlight(flightNum);
            Ticket ticket = bookingService.bookFlight(passenger, flight);
            if (ticket != null) {
                JOptionPane.showMessageDialog(frame, "Booking successful:\n" + ticket);
            } else {
                JOptionPane.showMessageDialog(frame, "Booking failed. Flight full or not found.");
            }
        });
  cancelButton.addActionListener(e -> {
            String ticketId = JOptionPane.showInputDialog("Enter Ticket ID to Cancel:");
            boolean success = bookingService.cancelTicket(ticketId);
            JOptionPane.showMessageDialog(frame, success ? "Ticket cancelled." : "Ticket not found.");
        });

        viewButton.addActionListener(e -> bookingService.viewAllBookings());

        frame.add(bookButton);
        frame.add(cancelButton);
        frame.add(viewButton);
        frame.setVisible(true);
    }
}
