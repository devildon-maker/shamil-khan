
using System;
using System.Collections.Generic;

public class Course
{
    public string Title { get; set; }
    public string Code { get; set; }
    public int Credits { get; set; }
    public List<Student> EnrolledStudents { get; set; }

    public Course(string title, string code, int credits)
    {
        Title = title;
        Code = code;
        Credits = credits;
        EnrolledStudents = new List<Student>();
    }

    public void DisplayInfo()
    {
        Console.WriteLine($"Course: {Title}, Code: {Code}, Credits: {Credits}");
        Console.WriteLine("Enrolled Students:");
        foreach (var student in EnrolledStudents)
        {
            Console.WriteLine($" - {student.Name} ({student.ID})");
        }
    }
}

public static class UniversityManagementSystem
{
    private static Dictionary<string, Course> coursesDictionary = new Dictionary<string, Course>();
    private static List<User> registeredUsers = new List<User>();
    private static User currentUser = null;

    public static void Main(string[] args)
    {
        bool running = true;

        while (running)
        {
            if (currentUser == null)
            {
                Console.WriteLine("\n--- University Management System ---");
                Console.WriteLine("1. Register");
                Console.WriteLine("2. Login");
                Console.WriteLine("3. Exit");
                Console.Write("Choose an option: ");

                try
                {
                    int choice = int.Parse(Console.ReadLine());
                    switch (choice)
                    {
                        case 1:
                            Register();
                            break;
                        case 2:
                            Login();
                            break;
                        case 3:
                            running = false;
                            break;
                        default:
                            Console.WriteLine("Invalid option. Try again.");
                            break;
                    }
                }
                catch (FormatException)
                {
                    Console.WriteLine("Invalid input. Please enter a number.");
                }
            }
            else
            {
                currentUser.DisplayMenu();
                HandleMenuInput(currentUser);
            }
        }
    }

    private static void HandleMenuInput(User user)
    {
        Console.Write("Select an option: ");

        try
        {
            int choice = int.Parse(Console.ReadLine());

            if (user is Admin)
            {
                switch (choice)
                {
                    case 1: AddCourse(); break;
                    case 2: UpdateCourse(); break;
                    case 3: DisplayAllCourses(); break;
                    case 4: SearchCourses(); break;
                    case 5: Logout(); break;
                    default: Console.WriteLine("Invalid option."); break;
                }
            }
            else if (user is Student)
            {
                switch (choice)
                {
                    case 1: SearchCourses(); break;
                    case 2: EnrollInCourse(user); break;
                    case 3: DisplayEnrolledCourses(user); break;
                    case 4: Logout(); break;
                    default: Console.WriteLine("Invalid option."); break;
                }
            }
        }
        catch (FormatException)
        {
            Console.WriteLine("Invalid input. Please enter a number.");
        }
    }

    private static void AddCourse()
    {
        Console.WriteLine("Enter Course Details:");
        Console.Write("Title: ");
        string title = Console.ReadLine();
        Console.Write("Code: ");
        string code = Console.ReadLine();
        Console.Write("Credits: ");
        int credits = int.Parse(Console.ReadLine());

        Course course = new Course(title, code, credits);
        coursesDictionary[code] = course; // Add to dictionary
        Console.WriteLine("Course added successfully.");
    }

    private static void UpdateCourse()
    {
        Console.Write("Enter Course Code to update: ");
        string code = Console.ReadLine();

        if (coursesDictionary.TryGetValue(code, out Course courseToUpdate))
        {
            Console.WriteLine("Enter new details for the course:");
            Console.Write("Title: ");
            string newTitle = Console.ReadLine();
            Console.Write("Credits: ");
            int newCredits = int.Parse(Console.ReadLine());

            courseToUpdate.Title = newTitle;
            courseToUpdate.Credits = newCredits;
            Console.WriteLine("Course details updated successfully.");
        }
        else
        {
            Console.WriteLine("Course not found.");
        }
    }

    private static void DisplayAllCourses()
    {
        Console.WriteLine("All courses in the university:");
        foreach (var course in coursesDictionary.Values)
        {
            course.DisplayInfo();
        }
    }

    private static void SearchCourses()
    {
        Console.Write("Enter course title to search: ");
        string searchString = Console.ReadLine();

        foreach (var course in coursesDictionary.Values)
        {
            if (course.Title.Contains(searchString, StringComparison.OrdinalIgnoreCase))
            {
                course.DisplayInfo();
            }
        }
    }

    private static void Register()
    {
        string id, name, password, role;

        do
        {
            Console.Write("Enter your ID: ");
            id = Console.ReadLine();
            if (string.IsNullOrWhiteSpace(id))
            {
                Console.WriteLine("ID cannot be empty. Please enter a valid ID.");
            }
        } while (string.IsNullOrWhiteSpace(id));

        User existingUser = registeredUsers.Find(user => user.ID == id);
        if (existingUser != null)
        {
            Console.WriteLine("A user with this ID already exists. Please try with a different ID.");
            return;
        }

        do
        {
            Console.Write("Enter your Name: ");
            name = Console.ReadLine();
            if (string.IsNullOrWhiteSpace(name))
            {
                Console.WriteLine("Name cannot be empty. Please enter a valid name.");
            }
        } while (string.IsNullOrWhiteSpace(name));

        do
        {
            Console.Write("Enter your Password: ");
            password = Console.ReadLine();
            if (string.IsNullOrWhiteSpace(password))
            {
                Console.WriteLine("Password cannot be empty. Please enter a valid password.");
            }
        } while (string.IsNullOrWhiteSpace(password));

        do
        {
            Console.Write("Enter your Role (Student/Admin): ");
            role = Console.ReadLine().ToLower();
            if (role != "student" && role != "admin")
            {
                Console.WriteLine("Invalid role. Please enter either Student or Admin.");
            }
        } while (role != "student" && role != "admin");

        User newUser = null;
        if (role == "admin")
        {
            newUser = new Admin(id, name, password);
        }
        else if (role == "student")
        {
            newUser = new Student(id, name, password);
        }

        registeredUsers.Add(newUser);
        Console.WriteLine("Registration successful! You can now log in.");
    }

    private static void Login()
    {
        Console.Write("Enter your ID: ");
        string id = Console.ReadLine();
        Console.Write("Enter your Password: ");
        string password = Console.ReadLine();

        currentUser = registeredUsers.Find(user => user.ID == id && user.Password == password);
        if (currentUser == null)
        {
            Console.WriteLine("Invalid ID or password. Please try again.");
        }
        else
        {
            Console.WriteLine($"Welcome, {currentUser.Name}!");
        }
    }

    private static void Logout()
    {
        currentUser = null;
        Console.WriteLine("You have logged out successfully.");
    }

    private static void EnrollInCourse(User user)
    {
        Console.Write("Enter the course code to enroll: ");
        string code = Console.ReadLine();

        if (coursesDictionary.TryGetValue(code, out Course courseToEnroll))
        {
            if (!courseToEnroll.EnrolledStudents.Contains(user as Student))
            {
                courseToEnroll.EnrolledStudents.Add(user as Student);
                Console.WriteLine($"You have successfully enrolled in '{courseToEnroll.Title}'.");
            }
            else
            {
                Console.WriteLine("You are already enrolled in this course.");
            }
        }
        else
        {
            Console.WriteLine("Course not found.");
        }
    }

    private static void DisplayEnrolledCourses(User user)
    {
        Console.WriteLine("Enrolled Courses:");
        foreach (var course in coursesDictionary.Values)
        {
            if (course.EnrolledStudents.Contains(user as Student))
            {
                course.DisplayInfo();
            }
        }
    }
}

public abstract class User
{
    public string ID { get; private set; }
    public string Name { get; private set; }
    public string Password { get; private set; }

    protected User(string id, string name, string password)
    {
        ID = id;
        Name = name;
        Password = password;
    }

    public abstract void DisplayMenu();
}

public class Admin : User
{
    public Admin(string id, string name, string password) : base(id, name, password) { }

    public override void DisplayMenu()
    {
        Console.WriteLine("\n--- Admin Menu ---");
        Console.WriteLine("1. Add Course");
        Console.WriteLine("2. Update Course");
        Console.WriteLine("3. Display All Courses");
        Console.WriteLine("4. Search Courses");
        Console.WriteLine("5. Logout");
    }
}

public class Student : User
{
    public Student(string id, string name, string password) : base(id, name, password) { }

    public override void DisplayMenu()
    {
        Console.WriteLine("\n--- Student Menu ---");
        Console.WriteLine("1. Search Courses");
        Console.WriteLine("2. Enroll in Course");
        Console.WriteLine("3. Display Enrolled Courses");
        Console.WriteLine("4. Logout");
    }
}
