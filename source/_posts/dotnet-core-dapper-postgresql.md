title: Use Dapper, .Net Core and Postgresql on Vagrant
date: 2016-10-25 20:42:47
tags:
  - dapper
  - dotnet-core
  - dotnet
  - postgresql

---

In this post you're going to learn how to setup a simple console application in
.Net core that talks to Postgresql.

This application will run on Ubuntu 14.04 using Vagrant.

I'm using iTerminal on macOS

## Index
- [Setting up Vagrant box.](#Setting_up_Vagrant_box)
- [Setting up Postgresql.](#Setting_up_Postgresql)
- [Creating .Net core console application.](#Creating_-Net_core_console_application-)
- [Adding Dapper and Npgsql](#Adding_Dapper_and_Npgsql)
- [Connecting to Database and persisting data.](#Connecting_to_Database_and_persisting_data-)


## Setting up Vagrant box

Create a new directory for our project:

```
> mkdir dapper-dotnet-postgresql && cd $_
> touch Vagrantfile
```

Use the following Vagrantfile. This Vagrant file will provision the box with 
`Postgresql` and `dotnet-dev-1.0.0-preview2-003131`. I got the installation steps for DotNet Core from
[Microsoft Net Core Website](https://www.microsoft.com/net/core#ubuntu)

```
## Vagrantfile
Vagrant.configure('2') do |config|
  config.vm.box = 'ubuntu/trusty64'
  config.ssh.forward_agent = true

  # Fix for: "stdin: is not a tty"
  # https://github.com/mitchellh/vagrant/issues/1673#issuecomment-28288042
  config.ssh.shell = %{bash -c 'BASH_ENV=/etc/profile exec bash'}

  config.vm.hostname = "dotnetcore.vm"
  config.vm.synced_folder ".", "/home/vagrant/src"
  config.vm.network :private_network, ip: "10.0.15.10"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end
  
  config.vm.provision "shell", inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    export LANGUAGE=en_US.UTF-8
    export LANG=en_US.UTF-8
    export LC_ALL=en_US.UTF-8
    locale-gen en_US.UTF-8
    dpkg-reconfigure locales
    
    sudo sh -c 'echo "deb [arch=amd64] https://apt-mo.trafficmanager.net/repos/dotnet-release/ trusty main" > /etc/apt/sources.list.d/dotnetdev.list'
    sudo apt-key adv --keyserver apt-mo.trafficmanager.net --recv-keys 417A0893 > /dev/null 2>&1

    sudo apt-get update -y -qq > /dev/null 2>&1

    sudo apt-get -y -q install dotnet-dev-1.0.0-preview2-003131 postgresql postgresql-contrib > /dev/null 2>&1
  SHELL
end

```

Now in your terminal window run:

```
> vagrant up && vagrant ssh
```

You can verify it worked by typing `dotnet` and `psql`:

```
vagrant@dotnetcore:~$ dotnet

Microsoft .NET Core Shared Framework Host....

vagrant@dotnetcore:~$ psql
psql: FATAL:  role "vagrant" does not exist
```

## Setting up Postgresql
Postgresql is a little different from Sql Server, here is a really basic breakdown 
on how it works:

- "Roles" are used in Postgres for Authentication and Authorization
- __ident__ identification is the default auth method. Similar to Active Directory.
- For Postgresql there are 2 user types: Linux ones and Postgres ones.
- You will need to create a user in both places in order to connect and query.
- By default there is going to be a "postgres" user, both in linux and postgres.

The following steps will help you on:

- Creating a user `dotnetcore_db_user`
- Becoming `postgres` user and adding `dotnetcore_db_user` as superuser
- Creating a sample database `dotnetcore_test`
- Adding a default password for your new user.
- Testing the connection.

```
vagrant@dotnetcore:~$ sudo adduser dotnetcore_db_user
Enter new UNIX password: password
Retype new UNIX password: password
##type Enter for all default values
vagrant@dotnetcore:~$ sudo -i -u postgres
postgres@dotnetcore:~$ createuser --interactive
$ Enter name of role to add: dotnetcore_db_user
$ Shall the new role be a superuser? (y/n) y
postgres@dotnetcore:~$ createdb dotnetcore_test

postgres@dotnetcore:~$ psql
postgres=# ALTER USER dotnetcore_db_user PASSWORD 'password';
postgres=# \q

postgres@dotnetcore:~$ exit
vagrant@dotnetcore:~$ sudo -i -u dotnetcore_db_user
dotnetcore_db_user@dotnetcore:~$ psql -d dotnetcore_test
dotnetcore_test=# \c
You are now connected to database "dotnetcore_test" as user "dotnetcore_db_user".

```

If you see that message, it means you're ready to create your first table inside `dotnetcore_test`, 
otherwise if __things go wrong__ you can check: not becomming `postgres` user before typing the commnads, 
not adding users in both places, not setting the password in postgresql for the given user.

Let's create a product table and insert one row into it:

```
dotnetcore_test=# CREATE TABLE products (
dotnetcore_test(# id serial not null PRIMARY KEY,
dotnetcore_test(# name varchar (50) NOT NULL,
dotnetcore_test(# color varchar (25) check (color in('red', 'blue', 'green')),
dotnetcore_test(# createdat timestamp without time zone default (now() at time zone 'utc'),
dotnetcore_test(# updatedat timestamp without time zone default (now() at time zone 'utc')
dotnetcore_test(# );
CREATE TABLE
dotnetcore_test=# INSERT INTO products (name, color) VALUES ('my first product', 'green');
INSERT 0 1
dotnetcore_test=# SELECT * FROM products;
type q to exit seeing all the rows of products.
dotnetcore_test=# \q
```

Now we can create our .Net Core console application inside our Vagrant box.

## Creating .Net core console application.

Make sure that you're back into `vagrant` home directory (`vagrant@dotnetcore:~$`) and from
there switch to `src` folder and create a new dotnet core application:

```
vagrant@dotnetcore:~$ cd ~ && cd src
vagrant@dotnetcore:~$ dotnet new
vagrant@dotnetcore:~/src$ dotnet restore
vagrant@dotnetcore:~/src$ dotnet run
```

After that last command you should see something like this:

```
Project src (.NETCoreApp,Version=v1.0) will be compiled because expected outputs are missing
Compiling src for .NETCoreApp,Version=v1.0

Compilation succeeded.
    0 Warning(s)
    0 Error(s)

Time elapsed 00:00:00.9978333


Hello World!
```

## Adding Dapper and Npgsql

Before connecting to our database, we need two nugget packages:

- Npgsql: Npgsql is the .NET data provider for PostgreSQL. 
It allows any program developed for .NET framework to access a PostgreSQL database server. [more](https://github.com/npgsql/npgsql)
- Dapper: Dapper - a simple object mapper for .Net. [more](https://github.com/StackExchange/dapper-dot-net)

To do this, modify your `project.json` file:

```
{
  "version": "1.0.0-*",
  "buildOptions": {
    "debugType": "portable",
    "emitEntryPoint": true
  },
  "dependencies": {},
  "frameworks": {
    "netcoreapp1.0": {
      "dependencies": {
        "Microsoft.NETCore.App": {
          "type": "platform",
          "version": "1.0.1"
        },
        "Dapper": "1.50.2",
        "Npgsql": "3.1.8"
      },
      "imports": "dnxcore50"
    }
  }
}

```

The added lines are below "Microsoft.NETCore.App" inside `dependencies`.

Go back to your command line and run:

```
vagrant@dotnetcore:~/src$ dotnet restore
vagrant@dotnetcore:~/src$ dotnet run

Hello World!
```

You haven't modified `Program.cs` until this point, but that's our next step, you
will connect to postgresql, insert a new row and then query that table.

## Connecting to Database and persisting data.

This will be a quick and dirty application, nothing production ready but it will
give a kick-start to something more substantial.

Start by modifying `Program.cs` file:

```
using System;
using System.Collections.Generic;
using Dapper;
using Npgsql;

namespace ConsoleApplication
{
    public class Program
    {
        public static void Main(string[] args)
        {
            Dapper.DefaultTypeMap.MatchNamesWithUnderscores = true;
            var connectionString = "Server=127.0.0.1;Port=5432;Database=dotnetcore_test;User Id=dotnetcore_db_user;Password=password";
            var insertCommand = "INSERT INTO products(name, color) VALUES(@name, @color)";
            var getAllCommand = "SELECT * FROM products;";
            

            using(var dbConnection = new NpgsqlConnection(connectionString))
            {
                dbConnection.Open();
                dbConnection.Query<Product>(insertCommand, new { name = "Test", color = "green" });
                var allProducts =  dbConnection.Query<Product>(getAllCommand);

                foreach(var product in allProducts)
                {
                    Console.WriteLine(product);
                }
                
            }

            Console.WriteLine("Press any key to exit");
            Console.ReadKey();
        }
    }

    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }

        public string Color { get; set; }

        public DateTime CreatedAt { get; set; }

        public DateTime UpdatedAt { get; set; }

        public override string ToString() 
        {
            return $"ID: {Id}, Name: {Name}, Color: {Color}, CreatedAt: {CreatedAt}, UpdatedAt: {UpdatedAt}";
        }
    }
}

```

Here's the breakdown of `Program.cs`:

- Created a `Product` class to map it to our table.
- `Dapper.DefaultTypeMap.MatchNamesWithUnderscores = true;` allows Dapper to map `created_at` and `updated_at` columns
- Declared required connection string and two sql commands.
- Then it opens a connection, insert a new row and queries for all rows.
- Print all rows to Console.

Once you have this, go back to your command line and type:

```
vagrant@dotnetcore:~/src$ dotnet run
```

If everything worked, you should see two entries in your table.


## Closing toughts

Creating .Net Core applications that connect to Postgresql seems pretty straightforward. As usual, 
setting up your environment with a new database is time consuming but hopefully you will only have
to make it one time.

Remember, __never har-dcode__ connection strings or passwords into your source code, I did it here
for the sake of this example.

Let me know if you have any problems while setting this up, and please let me know which topics of
.Net Core are important to you.





