<!---
{
  "id": "3b947ad2-5b1c-493b-ad08-364a61d5e6c0",
  "depends_on": ["d1bee1c7-d88a-4f00-a44e-3e402f6ee826"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-05-14",
  "keywords": ["MariaDB", "Docker", "Container", "Persistent Storage", "SQL"]
}
--->

# Using MariaDB in a Docker Container

> In this exercise you will learn how to deploy and interact with a MariaDB database using Docker containers. Furthermore we will explore how data persistence affects containerized databases and how to prevent data loss using volume mounts.

### Introduction

Containers offer an isolated and reproducible environment for running applications, and databases are no exception. MariaDB, a popular open-source relational database system, is well-suited for deployment within containers due to its lightweight nature and standardized configuration. In this exercise, we guide you through the lifecycle of a MariaDB container, highlighting key differences in container behavior with and without persistent storage.

We'll start by pulling the official MariaDB Docker image and running a container with default, ephemeral storage. You will connect to the database, create data, and observe what happens when the container is stopped and removed. The ephemeral nature of containers will become evident when the previously inserted data disappears upon recreation. Then, you'll rerun the container using a Docker volume to preserve your data across restarts and container deletions.

This hands-on practice helps you understand not just Docker commands and options but also how containerized databases behave in real-world deployment scenarios. These concepts are crucial when working in DevOps or backend development, where ensuring data persistence is essential for application reliability.

### Further Readings and Other Sources

* [MariaDB Docker Hub Repository](https://hub.docker.com/_/mariadb)
* [Docker Volumes Documentation](https://docs.docker.com/storage/volumes/)
* [SQL Tutorial for Beginners](https://www.w3schools.com/sql/)

### Tasks

1. **Pull the MariaDB Image:**
   Begin by ensuring you have the latest MariaDB image from Docker Hub. This guarantees compatibility with the latest features and security patches.

   ```bash
   docker pull mariadb
   ```

   Confirm the image is available locally:

   ```bash
   docker images | grep mariadb
   ```

2. **Run a MariaDB Container:** Launch a new MariaDB container named `mariadb-temp` using an environment variable to set the root password. This container will store data in its internal filesystem.

   ```bash
   docker run --name mariadb-temp -e MARIADB_ROOT_PASSWORD=my-secret-pw -d mariadb
   ```

   Check the container's status:

   ```bash
   docker ps -a
   ```

3. **Access the Container and MariaDB Shell:**
   Enter the running container using Bash, then access the MariaDB CLI:

   ```bash
   docker exec -it mariadb-temp bash
   mysql -u root -p
   ```

   Type the password `my-secret-pw` when prompted. You are now in the MariaDB command line.

4. **Create a Database, Table, and Insert Entries:**
   Inside the MariaDB shell, execute the following SQL statements step-by-step:

   ```sql
   CREATE DATABASE school;
   USE school;
   CREATE TABLE students (id INT PRIMARY KEY, name VARCHAR(100));
   INSERT INTO students VALUES (1, 'Alice');
   INSERT INTO students VALUES (2, 'Bob');
   INSERT INTO students VALUES (3, 'Charlie');
   SELECT * FROM students;
   ```

   This will create a new database, which is sort of an organizational structure, that can later be configured with fine-grained access-rights for specific users. We then tell the server, that we want to work on that database by using the `USE` command. This can be compared to moving into a specific working directory. In this database we create a table with fixed columns and add 3 entries. The `SELECT` statment then returns all columns from the table `students`.

   Verify all three entries are inserted correctly.

5. **Filter Table Content:**
   Use a SQL `LIKE` clause to filter students whose names start with 'A':

   ```sql
   SELECT * FROM students WHERE name LIKE 'A%';
   ```

   This should return only 'Alice'. We added a predicate, that generates an output for every entry in the table for which it evaluates to `true`.

6. **Exit, Stop and Remove the Container:**
   Exit the MariaDB shell and Bash shell, then stop and remove the container:

   ```bash
   exit    # to leave mysql
   exit    # to leave bash
   docker stop mariadb-temp
   docker rm mariadb-temp
   ```

   This step completely deletes the container and all of its data.

7. **Run a New Container:**
   Start a new container with the same configuration:

   ```bash
   docker run --name mariadb-temp -e MARIADB_ROOT_PASSWORD=my-secret-pw -d mariadb
   docker exec -it mariadb-temp bash
   mysql -u root -p
   ```

   Try running:

   ```sql
   SHOW DATABASES;
   ```

   You will notice the `school` database is gone. The data was only stored within the internal storage of the container. Is the container destroyed, the data is gone, even though the image still exists on the host machine. This illustrates that without persistent storage, data is not saved beyond the container's lifecycle.

8. **Use a Docker Volume for Persistence:**
   Create a named volume and rerun the MariaDB container using the volume to mount MariaDB's data directory:

   ```bash
   docker volume create mariadb-data
   docker run --name mariadb-persistent -e MARIADB_ROOT_PASSWORD=my-secret-pw \
     -v mariadb-data:/var/lib/mysql -d mariadb
   docker exec -it mariadb-persistent bash
   mysql -u root -p
   ```

   Repeat step 4 inside this session to create the database and insert entries. Then:

   ```bash
   exit
   docker stop mariadb-persistent
   docker rm mariadb-persistent
   ```

   Restart the container again with the same volume:

   ```bash
   docker run --name mariadb-persistent -e MARIADB_ROOT_PASSWORD=my-secret-pw \
     -v mariadb-data:/var/lib/mysql -d mariadb
   docker exec -it mariadb-persistent bash
   mysql -u root -p
   ```

   Run:

   ```sql
   USE school;
   SELECT * FROM students;
   ```

   This time the data is still available, demonstrating how persistent volumes safeguard container data. The Database-Files could also be backed up or moved to a different machine. 

### Questions

1. What happens to the MariaDB data when the container is deleted without persistent storage?
2. Why is using a volume critical for database containers?
3. How do you list all Docker volumes on your system?
4. Can you explain the command `-v mariadb-data:/var/lib/mysql` in detail?
5. What alternative methods besides named volumes could you use to persist data in Docker?

### Advice

Understanding container data persistence is key to managing reliable infrastructure. It’s tempting to treat containers as disposable, but databases must retain their state across restarts. Use this exercise to solidify your confidence with Docker's volume features. Practice mounting named volumes, inspecting them, and try extending this to other stateful services. You’ll often revisit these patterns when deploying applications in production or using orchestrators like Kubernetes. Don’t forget to explore how backups and migrations can be handled using similar volume-based strategies. This sheet is complemented well by the persistent storage module \[link-to-exercise-persistent-storage-101].
