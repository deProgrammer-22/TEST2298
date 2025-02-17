using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace Employee
{
    public class AppEmploy : DbContext
    {
        public AppEmploy(DbContextOptions<AppEmploy> options) : base(options) { }

        public DbSet<Employee> Employees { get; set; }
    }

    public class Employee
    {
        public int ID { get; set; }  
        public string? FirstName { get; set; }  
        public string? LastName { get; set; }  
        public string? Position { get; set; }  
    }

    public interface IEmployeeRepository
    {
        Task<Employee> CreateAsync(Employee employee);
        Task<Employee> GetAsync(int id);
        Task<List<Employee>> GetAllAsync(); 
        Task<Employee> UpdateAsync(int id, Employee employee);
        Task<bool> DeleteAsync(int id);
    }

    public class EmployeeRepository : IEmployeeRepository
    {
        private readonly AppEmploy _context;

        public EmployeeRepository(AppEmploy context)
        {
            _context = context;
        }

        public async Task<Employee> CreateAsync(Employee employee)
        {
            _context.Employees.Add(employee);
            await _context.SaveChangesAsync();
            return employee;
        }

        public async Task<Employee> GetAsync(int id)
        {
            var employee = await _context.Employees.FindAsync(id);
            if (employee == null)
            {
                throw new KeyNotFoundException($"Employee with ID {id} not found.");
            }
            return employee;
        }

        public async Task<List<Employee>> GetAllAsync() 
        {
            return await _context.Employees.ToListAsync();
        }

        public async Task<Employee> UpdateAsync(int id, Employee employee)
        {
            var existingEmployee = await _context.Employees.FindAsync(id);
            if (existingEmployee == null)
            {
                throw new KeyNotFoundException($"Employee with ID {id} not found.");
            }

            existingEmployee.FirstName = employee.FirstName ?? existingEmployee.FirstName;
            existingEmployee.LastName = employee.LastName ?? existingEmployee.LastName;
            existingEmployee.Position = employee.Position ?? existingEmployee.Position;

            await _context.SaveChangesAsync();
            return existingEmployee;
        }

        public async Task<bool> DeleteAsync(int id)
        {
            var employee = await _context.Employees.FindAsync(id);
            if (employee == null) return false;

            _context.Employees.Remove(employee);
            await _context.SaveChangesAsync();
            return true;
        }
    }

    public interface IEmployeeServices
    {
        Task<Employee> CreateEmployeeAsync(Employee employee);
        Task<Employee> GetEmployeeAsync(int id);
        Task<List<Employee>> GetAllEmployeesAsync();
        Task<Employee> UpdateEmployeeAsync(int id, Employee employee);
        Task<bool> DeleteEmployeeAsync(int id);
    }

    public class EmployeeService : IEmployeeServices
    {
        private readonly IEmployeeRepository _repository;

        public EmployeeService(IEmployeeRepository repository)
        {
            _repository = repository;
        }

        public async Task<Employee> CreateEmployeeAsync(Employee employee)
        {
            return await _repository.CreateAsync(employee);
        }

        public async Task<Employee> GetEmployeeAsync(int id)
        {
            return await _repository.GetAsync(id);
        }

        public async Task<List<Employee>> GetAllEmployeesAsync()
        {
            return await _repository.GetAllAsync();
        }

        public async Task<Employee> UpdateEmployeeAsync(int id, Employee employee)
        {
            return await _repository.UpdateAsync(id, employee);
        }

        public async Task<bool> DeleteEmployeeAsync(int id)
        {
            return await _repository.DeleteAsync(id);
        }
    }

    [ApiController]
    [Route("INPUT MO DITO/[controller]")]
    public class EmployeeController : ControllerBase
    {
        private readonly IEmployeeServices _employeeService;

        public EmployeeController(IEmployeeServices employeeService)
        {
            _employeeService = employeeService;
        }

        [HttpPost]
        public async Task<IActionResult> CreateEmployee([FromBody] Employee employee)
        {
            if (employee == null)
            {
                return BadRequest("Data is empty.");
            }
            var createdEmployee = await _employeeService.CreateEmployeeAsync(employee);
            return Ok(createdEmployee);
        }

        [HttpGet("{id}")]
        public async Task<IActionResult> GetEmployee(int id)
        {
            var employee = await _employeeService.GetEmployeeAsync(id);
            if (employee == null)
                return NotFound("Data not found.");
            return Ok(employee);
        }

        [HttpPut("{id}")]
        public async Task<IActionResult> UpdateEmployee(int id, [FromBody] Employee employee)
        {
            if (employee == null)
            {
                return BadRequest("Data is empty.");
            }

            var updatedEmployee = await _employeeService.UpdateEmployeeAsync(id, employee);
            if (updatedEmployee == null)
                return NotFound("Data not found.");
            return Ok(updatedEmployee);
        }

        [HttpDelete("{id}")]
        public async Task<IActionResult> DeleteEmployee(int id)
        {
            var result = await _employeeService.DeleteEmployeeAsync(id);
            if (!result)
                return NotFound("Data not found.");
            return Ok(result);
        }
    }
}
