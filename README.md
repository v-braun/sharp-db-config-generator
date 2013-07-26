# sharp-db-config-generator
> Generates a C# class from a SQL-Server Database Table

By [v-braun - www.dev-things.net](http://www.dev-things.net).

## Description
T4 based script that generates a C# configuration class from a Database Table.

The following table:
![table example](https://github.com/v-braun/sharp-db-config-generator/assets/master/media/db.jpg)

Results in this C# Class:

```csharp
namespace My.Default.Namespace {

using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;

	public class ApiConfiguration { 
		public SecurityConfiguration Security { get; private set; }

		#region constructor
		public ApiConfiguration(DataTable data){ 
			var config = TransformConfigTable(data);
			this.Security = new SecurityConfiguration();
			this.Security.LoggedUserSince = Get_DateTime(config, "Api.Security.LoggedUserSince");
 	
		}
		#endregion
 
		#region methods

        private IDictionary<string, string> TransformConfigTable(DataTable data) {
            var result = new Dictionary<string, string>();
            foreach (DataRow row in data.Rows) {
                var app = row["App"] as string;
                if (app != "WebApp")
                    continue;

                var group = row["Group"] as string;
                var key = row["Key"] as string;
                var value = row["Value"] as string;
                var configKey = $"{app}.{group}.{key}";
                if (value == null)
                    throw new ConstraintException($"Config value at path: {configKey} is null, all values should be provided in the db");

                result[configKey] = value;
            }

            return result;            
        }

        private string Get_string(IDictionary<string, string> data, string key) {
            if (!data.ContainsKey(key))
                throw new ConstraintException($"Config value at path: {key} does not exist");

            return data[key];
        }
        private int Get_int(IDictionary<string, string> data, string key) {
            var value = Get_string(data, key);
            if (string.IsNullOrEmpty(value))
                return default(int);

            int result;
            if (int.TryParse(value, out result))
                return result;
            else
                throw new ConstraintException($"Config value: {value} at path: {key} cannot be converted in to an int");

        }

        private decimal Get_decimal(IDictionary<string, string> data, string key) {
            var value = Get_string(data, key);
            if (string.IsNullOrEmpty(value))
                return default(int);

            decimal result;
            if (decimal.TryParse(value, out result))
                return result;
            else
                throw new ConstraintException($"Config value: {value} at path: {key} cannot be converted in to an decimal");
        }

        private DateTime Get_DateTime(IDictionary<string, string> data, string key) {
            var value = Get_string(data, key);
            if (string.IsNullOrEmpty(value))
                return DateTime.MinValue;

            DateTime result;
            if (DateTime.TryParse(value, out result))
                return result;
            else
                throw new ConstraintException($"Config value: {value} at path: {key} cannot be converted in to a DateTime");

        }
		#endregion
 
		#region config classes
		public class SecurityConfiguration { 
			public DateTime LoggedUserSince {get; set;}
		
		}

		#endregion
	}
}
```


## Usage
Copy&Paste the scripts

- DbConfigGenerator.tt
- MultipleOutputHelper.ttinclude

into your project.

Open `DbConfigGenerator.tt` and set the following parameters:

```csharp
const string DB_CONNECTION_STRING = @"Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=TestDb;Integrated Security=True";
const string DB_TABLE_NAME = "[dbo].[Configuration]";
const string CODE_DEFAULT_NAMESPACE = "My.Default.Namespace";
const string CODE_CLASS_POSTFIX = "Configuration";
```

After a Save of the file it generates your configuration classes.


### Known Issues

If you discover any bugs, feel free to create an issue on GitHub fork and
send me a pull request.

[Issues List](https://github.com/v-braun/sharp-db-config-generator/issues).

## Dependencies
- [MultipleOutputHelper](https://github.com/damieng/DamienGKit/blob/master/T4/MultipleOutputHelper/MultipleOutputHelper.ttinclude) from [Damien Guard](https://github.com/damieng)

## Authors

![image](https://avatars3.githubusercontent.com/u/4738210?v=3&s=50)  
[v-braun](https://github.com/v-braun/)



## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request


## License

See [LICENSE](https://github.com/v-braun/sharp-db-config-generator/blob/master/LICENSE).