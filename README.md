# workdayc-code

using System;

using System.Data.SqlClient;

using System.Net.Http;

using System.Text.Json;

using System.Threading.Tasks;

namespace WorkdayIntegration
{
    class Program
    {
        static async Task Main(string[] args)
        {
            // Workday API URL and authentication details
            
            string workdayApiUrl = "https://your-workday-url.com/api/v1/employees";
            
            string accessToken = "your-oauth-access-token";  // Assuming you have the OAuth token

            // SQL Server connection string
            string connectionString = "Server=your-server;Database=your-database;Integrated Security=True;";

            // Call Workday API to fetch employee data
            var employees = await FetchEmployeeDataFromWorkday(workdayApiUrl, accessToken);

            // Insert employee data into SQL database
            if (employees != null)
            {
                await InsertDataIntoSqlDatabase(employees, connectionString);
            }
            else
            {
                Console.WriteLine("No data fetched from Workday.");
            }
        }

        // Fetch employee data from Workday API
        static async Task<Employee[]> FetchEmployeeDataFromWorkday(string apiUrl, string token)
        {
            using (var client = new HttpClient())
            {
                client.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);
                var response = await client.GetStringAsync(apiUrl);

                // Parse the JSON response from Workday into Employee objects
                var employees = JsonSerializer.Deserialize<Employee[]>(response);

                return employees;
            }
        }

        // Insert employee data into SQL Server database
        static async Task InsertDataIntoSqlDatabase(Employee[] employees, string connectionString)
        {
            using (SqlConnection connection = new SqlConnection(connectionString))
            {
                await connection.OpenAsync();

                foreach (var employee in employees)
                {
                    // SQL query to insert employee data
                    string query = "INSERT INTO Employees (EmployeeId, FirstName, LastName, Email) VALUES (@EmployeeId, @FirstName, @LastName, @Email)";

                    using (SqlCommand command = new SqlCommand(query, connection))
                    {
                        command.Parameters.AddWithValue("@EmployeeId", employee.EmployeeId);
                        command.Parameters.AddWithValue("@FirstName", employee.FirstName);
                        command.Parameters.AddWithValue("@LastName", employee.LastName);
                        command.Parameters.AddWithValue("@Email", employee.Email);

                        await command.ExecuteNonQueryAsync();
                    }
                }

                Console.WriteLine("Data inserted successfully.");
            }
        }
    }

    // Employee class to deserialize JSON data
    public class Employee
    {
        public string EmployeeId { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public string Email { get; set; }
    }
}
