

using System.Data;
using MySql.Data;
using MySql.Data.MySqlClient;
class DataTier{
    public string connStr = "server=34.69.59.37;user=ooluwafemi;database=ooluwafemi;port=8080;password=ooluwafemi";

    // perform login check using Stored Procedure "LoginCount" in Database based on given user' studentID and Password
    public bool LoginCheck(User user){
        MySqlConnection conn = new MySqlConnection(connStr);
        try
        {  
            conn.Open();
            string procedure = "LoginCount";
            MySqlCommand cmd = new MySqlCommand(procedure, conn);
            cmd.CommandType = CommandType.StoredProcedure; // set the commandType as storedProcedure
            cmd.Parameters.AddWithValue("@inputUserID", user.userID);
            cmd.Parameters.AddWithValue("@inputUserPassword", user.userPassword);
            cmd.Parameters.Add("@userCount", MySqlDbType.Int32).Direction =  ParameterDirection.Output;
            MySqlDataReader rdr = cmd.ExecuteReader();
           
            int returnCount = (int) cmd.Parameters["@userCount"].Value;
            rdr.Close();
            conn.Close();

            if (returnCount ==1){
                return true;
            }
            else{
                return false;
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
            conn.Close();
            return false;
        }
       
    }

    // perform enrollment check using Stored Procedure "CheckEnrollment" based on user and semester
    public DataTable CheckEnrollment(User user){
        MySqlConnection conn = new MySqlConnection(connStr);
        Console.WriteLine("Please input a semester in TermYear format, e.g: Fall2022, Spring2021");
        string semester = Console.ReadLine();
        try
        {  
            conn.Open();
            string procedure = "CheckEnrollment";
            MySqlCommand cmd = new MySqlCommand(procedure, conn);
            cmd.CommandType = CommandType.StoredProcedure;
            cmd.Parameters.AddWithValue("@inputStudentID", user.userID);
            cmd.Parameters["@inputStudentID"].Direction = ParameterDirection.Input;
            cmd.Parameters.AddWithValue("@inputSemester", semester);
            cmd.Parameters["@inputSemester"].Direction = ParameterDirection.Input;

            MySqlDataReader rdr = cmd.ExecuteReader();

            DataTable tableEnrollment = new DataTable();
            tableEnrollment.Load(rdr);
            rdr.Close();
            conn.Close();
            return tableEnrollment;
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
            conn.Close();
            return null;
        }
    }

    public bool AddCourse(User user)
    {
        MySqlConnection conn = new MySqlConnection(connStr);
        Console.WriteLine("Please enter Course ID to add:");
        string courseID = Console.ReadLine();
        Console.WriteLine("Please enter semester (e.g. Fall2024):");
        string semester = Console.ReadLine();

        try
        {
            conn.Open();
            string procedure = "AddCourseEnrollment";
            MySqlCommand cmd = new MySqlCommand(procedure, conn);
            cmd.CommandType = CommandType.StoredProcedure;

            cmd.Parameters.AddWithValue("@inputStudentID", user.userID);
            cmd.Parameters.AddWithValue("@inputCourseID", courseID);
            cmd.Parameters.AddWithValue("@inputSemester", semester);

            int rowsAffected = cmd.ExecuteNonQuery();
            conn.Close();

            if (rowsAffected > 0)
            {
                Console.WriteLine(" Course added successfully!");
                return true;
            }
            else
            {
                Console.WriteLine(" Failed to add course. Please check input or existing enrollment.");
                return false;
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
            conn.Close();
            return false;
        }
    }

    // Drop a course for a student
    public bool DropCourse(User user)
    {
        MySqlConnection conn = new MySqlConnection(connStr);
        Console.WriteLine("Please enter Course ID to drop:");
        string courseID = Console.ReadLine();
        Console.WriteLine("Please enter semester (e.g. Fall2024):");
        string semester = Console.ReadLine();

        try
        {
            conn.Open();
            string procedure = "DropCourseEnrollment";
            MySqlCommand cmd = new MySqlCommand(procedure, conn);
            cmd.CommandType = CommandType.StoredProcedure;

            cmd.Parameters.AddWithValue("@inputStudentID", user.userID);
            cmd.Parameters.AddWithValue("@inputCourseID", courseID);
            cmd.Parameters.AddWithValue("@inputSemester", semester);

            int rowsAffected = cmd.ExecuteNonQuery();
            conn.Close();

            if (rowsAffected > 0)
            {
                Console.WriteLine(" Course dropped successfully!");
                return true;
            }
            else
            {
                Console.WriteLine(" Failed to drop course. Please check if you are enrolled.");
                return false;
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
            conn.Close();
            return false;
        }
    }

}


using System.Data;
using MySql.Data.MySqlClient;
class GuiTier{
    User user = new User();
    DataTier database = new DataTier();

    // print login page
    public User Login(){
        Console.WriteLine("------Welcome to Course Management System------");
        Console.WriteLine("Please input user ID (StudentID): ");
        user.userID = Convert.ToInt16(Console.ReadLine());
        Console.WriteLine("Please input password: ");
        user.userPassword = Console.ReadLine();
        return user;
    }
    // print Dashboard after user logs in successfully
    public int Dashboard(User user){
        DateTime localDate = DateTime.Now;
        Console.WriteLine("---------------Dashboard-------------------");
        Console.WriteLine($"Hello: {user.userID}; Date/Time: {localDate.ToString()}");
        Console.WriteLine("Please select an option to continue:");
        Console.WriteLine("1. Check Enrollment");
        Console.WriteLine("2. Add A Course");
        Console.WriteLine("3. Drop A Course");
        Console.WriteLine("4. Log Out");
        int option = Convert.ToInt16(Console.ReadLine());
        return option;
    }

    // show enrollment records returned from database
    public void DisplayEnrollment(DataTable tableEnrollment){
        Console.WriteLine("---------------Enrollment List-------------------");
        foreach(DataRow row in tableEnrollment.Rows){
           Console.WriteLine($"CourseID: {row["courseID"]} \t CourseName: {row["courseName"]} \t Semester:{row["semester"]}");
        }
    }

    public (string, string) GetCourseInput(string action)
    {
        Console.WriteLine($"Please enter Course ID to {action}:");
        string courseID = Console.ReadLine();
        Console.WriteLine("Please enter semester (e.g. Fall2024):");
        string semester = Console.ReadLine();
        return (courseID, semester);
    }

}
using System.Data;
using MySql.Data.MySqlClient;
class BusinessLogic
{
   
    static void Main(string[] args)
    {
        bool _continue = true;
        User user;
        GuiTier appGUI = new GuiTier();
        DataTier database = new DataTier();

        // start GUI
        user = appGUI.Login();

       
        if (database.LoginCheck(user)){

            while(_continue){
                int option  = appGUI.Dashboard(user);
                switch(option)
                {
                    // check enrollment
                    case 1:
                        DataTable tableEnrollment = database.CheckEnrollment(user);
                        if(tableEnrollment != null)
                            appGUI.DisplayEnrollment(tableEnrollment);
                        break;
                    // Add A Course
                    case 2:
                        database.AddCourse(user);
                        break;
                    // Drop A Course
                    case 3:
                        database.DropCourse(user);
                        break;
                    // Log Out
                    case 4:
                        _continue = false;
                        Console.WriteLine("Log out, Goodbye.");
                        break;
                    // default: wrong input
                    default:
                        Console.WriteLine("Wrong Input");
                        break;
                }

            }
        }
        else{
                Console.WriteLine("Login Failed, Goodbye.");
        }        
    }    
}


