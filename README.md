# EXP-3.1-JAVA

import java.io.*;
import java.sql.*;
import javax.servlet.*;
import javax.servlet.http.*;

public class WebAppServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        String action = request.getParameter("action");
        if (action == null) {
            out.println("<html><body><h2>Web Application Portal</h2>");
            out.println("<ul>");
            out.println("<li><a href='?action=login'>User Login</a></li>");
            out.println("<li><a href='?action=employees'>View Employees</a></li>");
            out.println("<li><a href='?action=attendance'>Student Attendance</a></li>");
            out.println("</ul></body></html>");
        } else if (action.equals("login")) {
            out.println("<html><body><h2>Login</h2>");
            out.println("<form method='post' action='?action=loginProcess'>");
            out.println("Username: <input type='text' name='username'><br><br>");
            out.println("Password: <input type='password' name='password'><br><br>");
            out.println("<input type='submit' value='Login'></form></body></html>");
        } else if (action.equals("employees")) {
            String empID = request.getParameter("EmpID");
            try {
                Class.forName("com.mysql.cj.jdbc.Driver");
                Connection con = DriverManager.getConnection("jdbc:mysql://localhost:3306/nimbusdb", "root", "root");
                Statement st = con.createStatement();
                String query = empID == null ? "SELECT * FROM Employee" : "SELECT * FROM Employee WHERE EmpID=" + empID;
                ResultSet rs = st.executeQuery(query);
                out.println("<html><body><h2>Employee Records</h2>");
                out.println("<form method='get' action=''><input type='hidden' name='action' value='employees'>");
                out.println("Search by ID: <input type='text' name='EmpID'><input type='submit' value='Search'></form>");
                out.println("<table border='1'><tr><th>EmpID</th><th>Name</th><th>Salary</th></tr>");
                while (rs.next()) out.println("<tr><td>" + rs.getInt(1) + "</td><td>" + rs.getString(2) + "</td><td>" + rs.getDouble(3) + "</td></tr>");
                out.println("</table><br><a href='?'>Home</a></body></html>");
                con.close();
            } catch (Exception e) {
                out.println("<h3>Error: " + e.getMessage() + "</h3>");
            }
        } else if (action.equals("attendance")) {
            out.println("<html><body><h2>Student Attendance</h2>");
            out.println("<form method='post' action='?action=attendanceSubmit'>");
            out.println("Student ID: <input type='text' name='StudentID'><br><br>");
            out.println("Date: <input type='date' name='Date'><br><br>");
            out.println("Status: <select name='Status'><option value='Present'>Present</option><option value='Absent'>Absent</option></select><br><br>");
            out.println("<input type='submit' value='Submit'></form><br><a href='?'>Home</a></body></html>");
        }
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        String action = request.getParameter("action");
        if ("loginProcess".equals(action)) {
            String user = request.getParameter("username");
            String pass = request.getParameter("password");
            if (user.equals("admin") && pass.equals("1234"))
                out.println("<html><body><h2>Welcome, " + user + "!</h2><a href='?'>Home</a></body></html>");
            else
                out.println("<html><body><h3>Invalid credentials. Try again.</h3><a href='?action=login'>Back</a></body></html>");
        } else if ("attendanceSubmit".equals(action)) {
            String sid = request.getParameter("StudentID");
            String date = request.getParameter("Date");
            String status = request.getParameter("Status");
            try {
                Class.forName("com.mysql.cj.jdbc.Driver");
                Connection con = DriverManager.getConnection("jdbc:mysql://localhost:3306/nimbusdb", "root", "root");
                PreparedStatement ps = con.prepareStatement("INSERT INTO Attendance (StudentID, Date, Status) VALUES (?, ?, ?)");
                ps.setString(1, sid);
                ps.setString(2, date);
                ps.setString(3, status);
                int i = ps.executeUpdate();
                if (i > 0)
                    out.println("<html><body><h3>Attendance Saved Successfully</h3><a href='?'>Home</a></body></html>");
                con.close();
            } catch (Exception e) {
                out.println("<h3>Error: " + e.getMessage() + "</h3>");
            }
        }
    }
}

