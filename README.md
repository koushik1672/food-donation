# food-donation
package food;
import javax.swing.*;
import javax.swing.border.EmptyBorder;
import javax.swing.event.MouseInputAdapter;
import java.awt.*;
import java.awt.event.MouseWheelEvent;
import java.sql.*;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Date;
import java.util.List;
import javax.swing.JTable;
import javax.swing.JScrollPane;
class DatabaseConnection {
private static final String URL = "jdbc:mysql://localhost:3306/food_donation_db";
private static final String USER = "root";
private static final String PASSWORD = "Srinivas@5566";
public static Connection getConnection() throws SQLException {
return DriverManager.getConnection(URL, USER, PASSWORD);
}
}
class User {
String name;
String address;
String phoneNumber;
String email;
List<String> donationHistory = new ArrayList<>();
public User(String name, String address, String phoneNumber, String email) {
this.name = name;
this.address = address;
this.phoneNumber = phoneNumber;
this.email = email;
}
public void saveToDatabase() {
try (Connection conn = DatabaseConnection.getConnection()) {
String query = "INSERT INTO Users (name, address, phoneNumber, email) VALUES (?,
14
?, ?, ?)";
PreparedStatement stmt = conn.prepareStatement(query,
Statement.RETURN_GENERATED_KEYS);
stmt.setString(1, name);
stmt.setString(2, address);
stmt.setString(3, phoneNumber);
stmt.setString(4, email);
stmt.executeUpdate();
ResultSet rs = stmt.getGeneratedKeys();
if (rs.next()) {
int userId = rs.getInt(1);
System.out.println("User ID generated: " + userId);
}
} catch (SQLException e) {
e.printStackTrace();
}
}
public void addDonationHistory(String donationDetails) {
donationHistory.add(donationDetails);
try (Connection conn = DatabaseConnection.getConnection()) {
String query = "INSERT INTO Donations (userId, donationDetails, donationDate)
VALUES ((SELECT id FROM Users WHERE email = ?), ?, ?)";
PreparedStatement stmt = conn.prepareStatement(query);
stmt.setString(1, email);
stmt.setString(2, donationDetails);
stmt.setTimestamp(3, new Timestamp(new Date().getTime()));
stmt.executeUpdate();
} catch (SQLException e) {
e.printStackTrace();
}
}
public void displayDonationHistory(JTextArea textArea) {
StringBuilder history = new StringBuilder();
try (Connection conn = DatabaseConnection.getConnection()) {
String query = "SELECT donationDetails FROM Donations WHERE userId = (SELECT id
FROM Users WHERE email = ?)";
PreparedStatement stmt = conn.prepareStatement(query);
stmt.setString(1, email);
ResultSet rs = stmt.executeQuery();
while (rs.next()) {
history.append(rs.getString("donationDetails")).append("\n");
}
} catch (SQLException e) {
e.printStackTrace();
}
if (history.length() == 0) {
textArea.setText("No donations made yet.");
15
} else {
textArea.setText(history.toString());
}
}
public String getContactInfo() {
return "Name: " + name + "\nAddress: " + address + "\nPhone Number: " +
phoneNumber + "\nEmail: " + email;
}
public void updateUserDetails(String newName, String newAddress, String
newPhoneNumber, String newEmail) {
try (Connection conn = DatabaseConnection.getConnection()) {
String query = "UPDATE Users SET name = ?, address = ?, phoneNumber = ?, email = ?
WHERE email = ?";
PreparedStatement stmt = conn.prepareStatement(query);
stmt.setString(1, newName);
stmt.setString(2, newAddress);
stmt.setString(3, newPhoneNumber);
stmt.setString(4, newEmail);
stmt.setString(5, this.email);
int updatedRows = stmt.executeUpdate();
if (updatedRows > 0) {
this.name = newName;
this.address = newAddress;
this.phoneNumber = newPhoneNumber;
this.email = newEmail;
System.out.println("User details updated successfully!");
} else {
System.out.println("Failed to update user details!");
}
} catch (SQLException e) {
e.printStackTrace();
}
}
public void deleteDonationHistory() {
try (Connection conn = DatabaseConnection.getConnection()) {
String query = "DELETE FROM Donations WHERE userId = (SELECT id FROM Users
WHERE email = ?)";
PreparedStatement stmt = conn.prepareStatement(query);
stmt.setString(1, email);
int deletedRows = stmt.executeUpdate();
if (deletedRows > 0) {
donationHistory.clear();
System.out.println("Donation history deleted successfully!");
} else {
System.out.println("No donation history found to delete.");
16
}
} catch (SQLException e) {
e.printStackTrace();
}
}
}
@SuppressWarnings("serial")
public class FoodDonationApp extends JFrame {
private User currentUser;
private boolean isDarkMode = true;
@SuppressWarnings("unused")
private List<String> foodBanks = Arrays.asList("The Akshaya Patra Foundation - Near
SRM University, Chennai", "Food Bank India - Potheri, Chennai");
@SuppressWarnings("unused")
private List<String> peopleInNeed = Arrays.asList("Ravi Kumar - Kattankulathur, SRM
Nagar", "Priya Devi - Potheri, Chennai");
private JPanel mainPanel;
private JButton toggleModeButton;
public FoodDonationApp() {
setTitle("Food Donation System");
setSize(700, 550);
setExtendedState(JFrame.MAXIMIZED_BOTH);
setDefaultCloseOperation(EXIT_ON_CLOSE);
setLocationRelativeTo(null);
mainPanel = new JPanel();
mainPanel.setLayout(new BorderLayout());
getContentPane().add(mainPanel);
JTabbedPane tabbedPane = new JTabbedPane();
tabbedPane.addTab("Register / Login", createRegisterPanel());
tabbedPane.addTab("Donate Food", createDonatePanel());
tabbedPane.addTab("Receive Food", createReceivePanel());
tabbedPane.addTab("View History", createHistoryPanel());
tabbedPane.addTab("Update Details", createUpdateDetailsPanel());
tabbedPane.addTab("Contact Info", createContactPanel());
mainPanel.add(tabbedPane, BorderLayout.CENTER);
toggleModeButton = new JButton("Toggle Dark Mode");
toggleModeButton.setToolTipText("Switch between Light and Dark Mode");
toggleModeButton.addActionListener(e -> toggleDarkMode());
mainPanel.add(toggleModeButton, BorderLayout.SOUTH);
applyDarkMode();
17
setVisible(true);
}
private JPanel createRegisterPanel() {
JPanel panel = new JPanel(new GridBagLayout());
GridBagConstraints gbc = new GridBagConstraints();
gbc.insets = new Insets(10, 10, 10, 10);
panel.setBorder(new EmptyBorder(20, 20, 20, 20));
panel.setBackground(Color.CYAN); // Added color to the panel
JTextField nameField = new JTextField(15);
JTextField addressField = new JTextField(15);
JTextField phoneField = new JTextField(15);
JTextField emailField = new JTextField(15);
gbc.gridx = 0; gbc.gridy = 0;
JLabel nameLabel = new JLabel("Name:");
nameLabel.setForeground(Color.BLACK); // Set label color to black
panel.add(nameLabel, gbc);
gbc.gridx = 1; gbc.gridy = 0; panel.add(nameField, gbc);
gbc.gridx = 0; gbc.gridy = 1;
JLabel addressLabel = new JLabel("Address:");
addressLabel.setForeground(Color.BLACK); // Set label color to black
panel.add(addressLabel, gbc);
gbc.gridx = 1; gbc.gridy = 1; panel.add(addressField, gbc);
gbc.gridx = 0; gbc.gridy = 2;
JLabel phoneLabel = new JLabel("Phone Number:");
phoneLabel.setForeground(Color.BLACK); // Set label color to black
panel.add(phoneLabel, gbc);
gbc.gridx = 1; gbc.gridy = 2; panel.add(phoneField, gbc);
gbc.gridx = 0; gbc.gridy = 3;
JLabel emailLabel = new JLabel("Email:");
emailLabel.setForeground(Color.BLACK); // Set label color to black
panel.add(emailLabel, gbc);
gbc.gridx = 1; gbc.gridy = 3; panel.add(emailField, gbc);
JButton registerButton = new JButton("Register / Login");
registerButton.setBackground(Color.GREEN); // Added color to button
gbc.gridx = 1; gbc.gridy = 4; panel.add(registerButton, gbc);
registerButton.addActionListener(e -> {
String name = nameField.getText();
18
String address = addressField.getText();
String phone = phoneField.getText();
String email = emailField.getText();
if (name.isEmpty() || address.isEmpty() || phone.isEmpty() || email.isEmpty()) {
JOptionPane.showMessageDialog(this, "All fields are required!", "Error",
JOptionPane.ERROR_MESSAGE);
} else {
currentUser = new User(name, address, phone, email);
currentUser.saveToDatabase();
JOptionPane.showMessageDialog(this, "User registration/login successful!");
}
});
return panel;
}
private JPanel createDonatePanel() {
JPanel panel = new JPanel(new GridBagLayout());
GridBagConstraints gbc = new GridBagConstraints();
gbc.insets = new Insets(10, 10, 10, 10);
panel.setBorder(new EmptyBorder(20, 20, 20, 20));
panel.setBackground(Color.LIGHT_GRAY); // Added color to the panel
// Create text fields for food name, quantity, and location
JTextField foodNameField = new JTextField(15);
JTextField quantityField = new JTextField(15);
JTextField locationField = new JTextField(15);
gbc.gridx = 0; gbc.gridy = 0;
JLabel foodNameLabel = new JLabel("Food Name:");
foodNameLabel.setForeground(Color.BLACK); // Set label color to black
panel.add(foodNameLabel, gbc);
gbc.gridx = 1; gbc.gridy = 0;
panel.add(foodNameField, gbc); // Add food name field
gbc.gridx = 0; gbc.gridy = 1;
JLabel quantityLabel = new JLabel("Quantity:");
quantityLabel.setForeground(Color.BLACK);
panel.add(quantityLabel, gbc);
gbc.gridx = 1; gbc.gridy = 1;
panel.add(quantityField, gbc); // Add quantity field
gbc.gridx = 0; gbc.gridy = 2;
JLabel locationLabel = new JLabel("Location:");
locationLabel.setForeground(Color.BLACK);
panel.add(locationLabel, gbc);
19
gbc.gridx = 1; gbc.gridy = 2;
panel.add(locationField, gbc); // Add location field
JButton donateButton = new JButton("Donate");
donateButton.setBackground(Color.MAGENTA); // Added color to button
gbc.gridx = 0; gbc.gridy = 3; gbc.gridwidth = 2; // Center the button
panel.add(donateButton, gbc);
donateButton.addActionListener(e -> {
String foodName = foodNameField.getText();
String quantity = quantityField.getText();
String location = locationField.getText();
if (foodName.isEmpty() || quantity.isEmpty() || location.isEmpty()) {
JOptionPane.showMessageDialog(this, "Please enter food name, quantity, and
location.", "Error", JOptionPane.ERROR_MESSAGE);
} else {
// Combine input details into one string
String donationDetails = "Food: " + foodName + " | Quantity: " + quantity + " |
Location: " + location;
currentUser.addDonationHistory(donationDetails);
JOptionPane.showMessageDialog(this, "Donation recorded successfully!");
foodNameField.setText(""); // Clear food name field
quantityField.setText(""); // Clear quantity field
locationField.setText(""); // Clear location field
}
});
return panel;
}
private JPanel createReceivePanel() {
// Create a new panel with BorderLayout
JPanel panel = new JPanel(new BorderLayout());
// Column Names
String[] columnNames = {"City", "Food Bank Name", "Mobile Number"};
// Food Bank Data
Object[][] data = {
{"Amaravathi", "Amaravati Food Bank", "+91 98765 43210"},
{"Visakhapatnam", "Visakhapatnam Food Bank", "+91 99876 54321"},
{"Vijayawada", "Vijayawada Food Bank", "+91 91234 56789"},
{"Guntur", "Guntur Food Bank", "+91 99888 77665"},
{"Tirupati", "Tirupati Food Bank", "+91 87654 32109"},
{"Nellore", "Nellore Food Bank", "+91 94567 89012"},
{"Kurnool", "Kurnool Food Bank", "+91 93456 78901"},
{"Rajahmundry", "Rajahmundry Food Bank", "+91 92345 67890"},
20
{"Anantapur", "Anantapur Food Bank", "+91 91012 34567"},
{"Chittoor", "Chittoor Food Bank", "+91 98987 65432"},
{"Hyderabad", "Hyderabad Food Bank", "+91 90000 11111"},
{"Warangal", "Warangal Food Bank", "+91 99888 77777"},
{"Nizamabad", "Nizamabad Food Bank", "+91 98888 88888"},
{"Khammam", "Khammam Food Bank", "+91 98765 67890"},
{"Karimnagar", "Karimnagar Food Bank", "+91 97654 32109"},
{"Mahbubnagar", "Mahbubnagar Food Bank", "+91 96543 21098"},
{"Ranga Reddy", "Ranga Reddy Food Bank", "+91 95432 10987"},
{"Adilabad", "Adilabad Food Bank", "+91 94321 09876"},
{"Medak", "Medak Food Bank", "+91 93210 98765"},
{"Medchal-Malkajgiri", "Medchal Food Bank", "+91 92109 87654"},
{"Chennai", "Chennai Food Bank", "+91 95000 22222"},
{"Coimbatore", "Coimbatore Food Bank", "+91 94000 33333"},
{"Madurai", "Madurai Food Bank", "+91 93000 44444"},
{"Tiruchirappalli", "Tiruchirappalli Food Bank", "+91 92000 55555"},
{"Salem", "Salem Food Bank", "+91 91000 66666"},
{"Erode", "Erode Food Bank", "+91 90000 77777"},
{"Tirunelveli", "Tirunelveli Food Bank", "+91 89000 88888"}
};
// Create JTable with data
JTable table = new JTable(data, columnNames);
// Enable sorting on the table
table.setAutoCreateRowSorter(true);
// Set column widths for better readability
table.getColumnModel().getColumn(0).setPreferredWidth(100); // City
table.getColumnModel().getColumn(1).setPreferredWidth(200); // Food Bank Name
table.getColumnModel().getColumn(2).setPreferredWidth(150); // Mobile Number
// Set panel and table colors for light mode
panel.setBackground(Color.WHITE); // Set panel background to white
table.setBackground(Color.LIGHT_GRAY); // Set table background to light gray
table.setForeground(Color.BLACK); // Set table foreground to black
// Add Mouse Listener for Row Click Events
table.addMouseListener(new MouseInputAdapter() {
public void mouseClicked(MouseEvent e) {
int row = table.getSelectedRow();
String city = (String) table.getValueAt(row, 0);
String foodBankName = (String) table.getValueAt(row, 1);
String mobileNumber = (String) table.getValueAt(row, 2);
// Display selected food bank details
JOptionPane.showMessageDialog(panel, "Contact " + foodBankName + " in " + city
+ ": " + mobileNumber);
}
});
21
// Wrap the JTable in a JScrollPane
JScrollPane scrollPane = new JScrollPane(table);
panel.add(scrollPane, BorderLayout.CENTER); // Add scroll pane to panel
return panel; // Return the completed panel
}
private JPanel createHistoryPanel() {
JPanel panel = new JPanel(new BorderLayout());
JTextArea historyArea = new JTextArea();
historyArea.setEditable(false);
JScrollPane scrollPane = new JScrollPane(historyArea);
panel.add(scrollPane, BorderLayout.CENTER);
JButton viewHistoryButton = new JButton("View Donation History");
viewHistoryButton.setBackground(Color.ORANGE); // Added color to button
panel.add(viewHistoryButton, BorderLayout.SOUTH);
viewHistoryButton.addActionListener(e -> {
if (currentUser != null) {
currentUser.displayDonationHistory(historyArea);
} else {
JOptionPane.showMessageDialog(this, "Please register or log in first.", "Error",
JOptionPane.ERROR_MESSAGE);
}
});
return panel;
}
private JPanel createUpdateDetailsPanel() {
JPanel panel = new JPanel(new GridBagLayout());
GridBagConstraints gbc = new GridBagConstraints();
gbc.insets = new Insets(10, 10, 10, 10);
panel.setBorder(new EmptyBorder(20, 20, 20, 20));
panel.setBackground(Color.LIGHT_GRAY); // Added color to the panel
JTextField nameField = new JTextField(15);
JTextField addressField = new JTextField(15);
JTextField phoneField = new JTextField(15);
JTextField emailField = new JTextField(15);
gbc.gridx = 0; gbc.gridy = 0;
JLabel nameLabel = new JLabel("New Name:");
nameLabel.setForeground(Color.BLACK); // Set label color to black
panel.add(nameLabel, gbc);
22
gbc.gridx = 1; gbc.gridy = 0; panel.add(nameField, gbc);
gbc.gridx = 0; gbc.gridy = 1;
JLabel addressLabel = new JLabel("New Address:");
addressLabel.setForeground(Color.BLACK); // Set label color to black
panel.add(addressLabel, gbc);
gbc.gridx = 1; gbc.gridy = 1; panel.add(addressField, gbc);
gbc.gridx = 0; gbc.gridy = 2;
JLabel phoneLabel = new JLabel("New Phone Number:");
phoneLabel.setForeground(Color.BLACK); // Set label color to black
panel.add(phoneLabel, gbc);
gbc.gridx = 1; gbc.gridy = 2; panel.add(phoneField, gbc);
gbc.gridx = 0; gbc.gridy = 3;
JLabel emailLabel = new JLabel("New Email:");
emailLabel.setForeground(Color.BLACK); // Set label color to black
panel.add(emailLabel, gbc);
gbc.gridx = 1; gbc.gridy = 3; panel.add(emailField, gbc);
JButton updateButton = new JButton("Update Details");
updateButton.setBackground(Color.YELLOW); // Added color to button
gbc.gridx = 1; gbc.gridy = 4; panel.add(updateButton, gbc);
updateButton.addActionListener(e -> {
if (currentUser != null) {
String newName = nameField.getText();
String newAddress = addressField.getText();
String newPhone = phoneField.getText();
String newEmail = emailField.getText();
if (newName.isEmpty() || newAddress.isEmpty() || newPhone.isEmpty() ||
newEmail.isEmpty()) {
JOptionPane.showMessageDialog(this, "All fields are required!", "Error",
JOptionPane.ERROR_MESSAGE);
} else {
currentUser.updateUserDetails(newName, newAddress, newPhone, newEmail);
JOptionPane.showMessageDialog(this, "User details updated successfully!");
}
} else {
JOptionPane.showMessageDialog(this, "Please register or log in first.", "Error",
JOptionPane.ERROR_MESSAGE);
}
});
return panel;
23
}
private JPanel createContactPanel() {
JPanel panel = new JPanel(new BorderLayout());
JTextArea contactArea = new JTextArea();
contactArea.setEditable(false);
JScrollPane scrollPane = new JScrollPane(contactArea);
panel.add(scrollPane, BorderLayout.CENTER);
JButton viewContactButton = new JButton("View Contact Info");
viewContactButton.setBackground(Color.PINK); // Added color to button
panel.add(viewContactButton, BorderLayout.SOUTH);
viewContactButton.addActionListener(e -> {
if (currentUser != null) {
contactArea.setText(currentUser.getContactInfo());
} else {
JOptionPane.showMessageDialog(this, "Please register or log in first.", "Error",
JOptionPane.ERROR_MESSAGE);
}
});
return panel;
}
private void toggleDarkMode() {
isDarkMode = !isDarkMode;
applyDarkMode();
}
private void applyDarkMode() {
if (isDarkMode) {
mainPanel.setBackground(Color.DARK_GRAY);
toggleModeButton.setForeground(Color.WHITE);
toggleModeButton.setBackground(Color.GRAY);
} else {
mainPanel.setBackground(Color.WHITE);
toggleModeButton.setForeground(Color.BLACK);
toggleModeButton.setBackground(Color.LIGHT_GRAY);
}
SwingUtilities.updateComponentTreeUI(this);
}
public static void main(String[] args) {
SwingUtilities.invokeLater(FoodDonationApp::new);
}
}
24
Code for backend
-- **1. User Management Module**
CREATE TABLE Users (
user_id INT PRIMARY KEY AUTO_INCREMENT,
name VARCHAR(100) NOT NULL,
email VARCHAR(100) UNIQUE NOT NULL,
password VARCHAR(255) NOT NULL,
phone VARCHAR(15),
address VARCHAR(255),
role ENUM('donor', 'receiver', 'admin') DEFAULT 'receiver',
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
-- **2. Donation Management Module**
CREATE TABLE Donations (
donation_id INT PRIMARY KEY AUTO_INCREMENT,
donor_id INT,
food_type VARCHAR(100) NOT NULL,
quantity INT NOT NULL,
pickup_location VARCHAR(255) NOT NULL,
expiration_date DATE,
status ENUM('available', 'received') DEFAULT 'available',
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
FOREIGN KEY (donor_id) REFERENCES Users(user_id)
);
-- **3. Receiving Management Module**
CREATE TABLE Received_Donations (
receive_id INT PRIMARY KEY AUTO_INCREMENT,
receiver_id INT,
donation_id INT,
received_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
FOREIGN KEY (receiver_id) REFERENCES Users(user_id),
FOREIGN KEY (donation_id) REFERENCES Donations(donation_id)
);
-- **4. Google Maps Integration Module**
-- No specific table needed, managed with Google Maps API integration.
-- **5. History and Tracking Module**
CREATE TABLE Donation_History (
history_id INT PRIMARY KEY AUTO_INCREMENT,
donation_id INT,
donor_id INT,
receiver_id INT,
food_type VARCHAR(100),
25
quantity INT,
donation_date TIMESTAMP,
FOREIGN KEY (donation_id) REFERENCES Donations(donation_id),
FOREIGN KEY (donor_id) REFERENCES Users(user_id),
FOREIGN KEY (receiver_id) REFERENCES Users(user_id)
);
-- **6. Notification and Communication Module**
CREATE TABLE Notifications (
notification_id INT PRIMARY KEY AUTO_INCREMENT,
user_id INT,
message TEXT,
is_read BOOLEAN DEFAULT FALSE,
sent_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
FOREIGN KEY (user_id) REFERENCES Users(user_id)
);
-- **7. Feedback and Rating Module**
CREATE TABLE Feedback (
feedback_id INT PRIMARY KEY AUTO_INCREMENT,
user_id INT,
donation_id INT,
rating INT CHECK (rating BETWEEN 1 AND 5),
comments TEXT,
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
FOREIGN KEY (user_id) REFERENCES Users(user_id),
FOREIGN KEY (donation_id) REFERENCES Donations(donation_id)
);
-- **8. Admin Management Module**
-- Admin users are included in the Users table with the 'role' column set to 'admin'.
-- **9. UI and UX Module**
-- No SQL required, handled through frontend.
-- **10. Reporting and Analytics Module**
-- Query to retrieve total donations made:
SELECT COUNT(*) AS total_donations FROM Donations;
-- Query to retrieve most popular food type:
SELECT food_type, COUNT(*) AS count
FROM Donations
GROUP BY food_type
ORDER BY count DESC
LIMIT 1;
-- Query to retrieve the number of users by role:
SELECT role, COUNT(*) AS user_count
26
FROM Users
GROUP BY role;
-- Query to retrieve feedback summary for each donation:
SELECT donation_id, AVG(rating) AS avg_rating, COUNT(*) AS feedback_count
FROM Feedback
GROUP BY donation_id;
