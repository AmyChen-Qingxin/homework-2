# Vanderbilt - Big Data 2020 - Homework 2

**Due Date  19 February 2020: 11:59 CT**

* Set up and upload MLB Data in to MongoDB instance running on EC2 (described below).
* Query and analyze MLB data 
# Useful Information

## Example

Take a look at the example mondb notebook in [mongo-notebook-example](mongo-notebook-example) folder. To actually run it - you will need some credentials. It will be emailed to the class and will be valid for two weeks.

## Mongodb Reference

Take a look at the following piazza post: https://piazza.com/class/k51vgkts30b12t?cid=72
Also useful the following [slides ](https://github.com/vu-bigdata-2020/lectures/blob/master/05_nosql/NoSQLMongoDB.pdf) from lecture notes in class.


## Mongodb Compass

You can use the equivalent of SQl workbench to work with your mongodb database. It is available at https://www.mongodb.com/products/compass

## Important

Read the instructions at https://github.com/vu-bigdata-2020/lectures/blob/master/00-assignmentInstructions/AcceptingaGithubassignment.pdf


## Assignment Repository

Repositories will be created for each student. You should see yours at 

    https://github.com/vu-bigdata-2020/homework-2-<GITHUB USERNAME> 

Clone the repository to your home directory on the cluster using:

    git clone https://github.com/vu-bigdata-2020/homework-2-<GITHUB USERNAME>.git

I may push updates to this homework assignment in the future. To setup an upstream repo, do the following:

    git remote add upstream https://github.com/vu-bigdata-2020/homework-2.git

To pull updates do the following:
    
    git fetch upstream
    git merge upstream/master

You will need to resolve conflicts if they occur. 

## Github

To push code to your repo use the git commit and push commands. But first set some settings:

	git config --global user.name "Your Name"
	git config --global user.email you@example.com

Once you modify files, use git's add, commit and push commands to push files to your repo. 

	git add file.txt
	git commit -a -m 'commit message'
	git push origin master

If you would like to use SSH keys on Github, follow the instructions at:

	https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/0

## Updating the python Notebook URL after accepting the assignment on classroom.github.com

Once you have accepted the assignment, upate all notebooks (*.ipynb) including any example I provided as discussed in in https://github.com/vu-bigdata-2020/lectures/blob/master/00-assignmentInstructions/AcceptingaGithubassignment.pdf

## AWS
To access AWS go to https://aws.amazon.com/education/awseducate/ and use the account you created when you were invited to the class. Ensure that you can access this account and can land into an AWS console as shown below.

# Assignment 

## Step-1 Create the EC2 Instance

First install the AWS EC2 using the AWS link below (use AWS free educate account login)

Note : In Step 3 :-Configure your instance . Choose Ubuntu server instead of Amazon linux.
Note : Dont do Step 5 as it will terminate your AWS EC2 instance
https://aws.amazon.com/getting-started/tutorials/launch-a-virtual-machine/?trk=gs_card&e=gs&p=gsrc

Caution: After doing your assignment make sure to shut down the EC2 instance and logout. This is necessary to avoid unnecessary charging to your AWS account.
Follow the instructions carefully to remain within **free tier**. That last part is very important.

So after installing Ubuntu server and connecting to Git bash here, We get a prompt as below -


** Note ** open the security group to allow incoming connections from anywhere on port 27017. You did this in previous assignment for MYSQL. It will work similarly here. See  this [PDF file for instructions](https://github.com/vu-bigdata-2020/lectures/blob/master/00-aws-setup-guide/Guide%20to%20use%20two%20EC2%20instances.pdf).

## Step-2 Install the MongoDb packages

Login to your EC2 then type the commands:

Import the public key used for accessing package management system
	
	$wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -

Create a list file for mongoDB
	
	$echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.2.list

	$sudo apt-get update


	$sudo apt-get install -y mongodb-org

Start the mongodb:

	$sudo service mongod start

Verify the mongod service
	
	$sudo service mongod status
	
## Step-3 Enable remote access to the mongoDB server running on EC2

Follow the instruction in the below link:

Create the remote users, but first create admin -
Enter the mongo shell on EC2
	$sudo Mongo
Select admin DB
	>use admin:

** change the admin password to something else **

Create the “admin” user (you can call it whatever you want)

	> db.createUser({ user: "admin", pwd: "adminpassword", roles: [{ role: "userAdminAnyDatabase", db: "admin" }] })
	> db.auth("admin", "adminpassword")

We are now going to enable authentication on the MongoDB instance, by modifying the mongod.conf file. If you’re on Linux:

	$sudo vim  /etc/mongod.conf

Add these lines at the bottom of the YAML config file:
security:
    authorization: enabled

This will enable authentication on your database instance. 


Now restart the mongod service (Ubuntu syntax) for the changes to take effect.
	$sudo service mongod restart
You can check if the service is up with:
	$sudo service mongod status

To create a external user login to mongo db account such as 'ubuntu'- 
Now login to mongo shell and select admin db and authenticate

	$sudo mongo

	>use admin
	>db.auth("admin", "adminpassword")

now create lahman database in mongo
	>use  lahman;

create remote user name - 'ubuntu' and a passowrd who can use lahman db 	

	>db.createUser({ user: "ubuntu", pwd: "yourpassword", roles: [{ role: "dbOwner", db: "lahman" }] })

Check that everything went fine by trying to authenticate, with the db.auth(user, pwd) function.

	>db.auth("ubuntu", "yourpassword")
  
 ** Note ** - keep your username and password private. Very important. This is what you will use to connect to the database. 

Refer to the link if you get stuck:
https://medium.com/@matteocontrini/how-to-setup-auth-in-mongodb-3-0-properly-86b60aeef7e8


## Step-4 loading the MongoDB with Lahman database

Download the lahman database to your windows or Mac Host  from http://www.seanlahman.com/files/database/
Use the lahman_sql_2012 comma delimited version (CSV) files. 

Download the lahman_sql_2012 database into your host (MAC or Windows) and then scp it into the EC2 ubuntu server using below cmd: 

	$scp  -i "<path to your pem file>.pem"   <path to lahman_2012.csv files> <username>@<ec2-name>:/home/ubuntu/
  
 **Note** you can also use curl or wget to directly copy from the web source 

login to the EC2 server and go to /home/ubuntu.
	
	$cd /home/ubuntu
	
Then import the csv files into mongoDB using the below command.
Do this for all the .csv files

	$mongoimport -d <dbmane> -c <collection_name>t --type csv --file <input.csv> --headerline.

Below are the import commands for all csv files to import into the mongodb - **you need to update the username and password to what you set up -- see the instructions above**

	$mongoimport -d lahman -c AllstarFull --type csv --file AllstarFull.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c AwardsSharePlayers --type csv --file AwardSharePlayers.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c Appearances --type csv --file Appearances.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c AwardManagers --type csv --file AwardManagers.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c AwardShareManagers --type csv --file AwardShareManagers.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c AwardPlayers --type csv --file AwardPlayers.csv --headerline --username "ubuntu" --password "yourpassword"

 	$mongoimport -d lahman -c Batting --type csv --file Batting.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c BattingPost --type csv --file BattingPost.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c Fielding --type csv --file Fielding.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c FieldingOF --type csv --file FieldingOF.csv --headerline --username "ubuntu" --password "yourpassword"
 
 	$mongoimport -d lahman -c FieldingPost --type csv --file FieldingPost.csv --headerline --username "ubuntu" --password "yourpassword"
 	4mongoimport -d lahman -c HallOfFame --type csv --file HallOfFame.csv --headerline --username "ubuntu" --password "yourpassword"
 
 	$mongoimport -d lahman -c Managers --type csv --file Managers.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c ManagersHalf --type csv --file ManagersHalf.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c Master --type csv --file Master.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c Pitching --type csv --file Pitching.csv --headerline --username "ubuntu" --password "yourpassword"
 
 	$mongoimport -d lahman -c PitchingPost --type csv --file PitchingPost.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c Salaries --type csv --file Salaries.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c Schools --type csv --file Schools.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c SchoolsPlayers --type csv --file SchoolsPLayers.csv --headerline --username "ubuntu" --password "yourpassword"
 
 	$mongoimport -d lahman -c SeriesPost --type csv --file SeriesPost.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c Teams --type csv --file Teams.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c TeamsFranchises --type csv --file TeamFranchises.csv --headerline --username "ubuntu" --password "yourpassword"
 	$mongoimport -d lahman -c TeamsHalf --type csv --file TeamsHalf.csv --headerline --username "ubuntu" --password "yourpassword"



## Step 5- Check initial Colab Connection

Run the Colab connection script [test-colab-mongodb.ipynb](test-colab-mongodb.ipynb) and ensure that you get the connection and the number of tables correctly. Make sure that you update the database name, the username and the password. 

**Remember** to update the python notebook as discussed in https://github.com/vu-bigdata-2020/lectures/blob/master/00-assignmentInstructions/AcceptingaGithubassignment.pdf

Remember to shutoff the EC2 instance when you are not using it.

## Step 6 -  Queries [80 points]

Implement a function per query hw2.ipynb. Record the answers there and save it back to your repository.

**Remember** to update the python notebook as discussed in https://github.com/vu-bigdata-2020/lectures/blob/master/00-assignmentInstructions/AcceptingaGithubassignment.pdf


The queries are

1. The number of all stars in allstarfull.
2. The most home runs in a season by a single player (using the batting table).
3. The playerid of the player with the most home runs in a season.
4. The number of leagues in the batting table.
5. Barry Bond's average batting average (playerid = 'bondsba01') where batting average is hits / at-bats. Note you will nead to cast hits to get a decimal: cast(h as real)
6. The teamid with the fewest hits in the year 2000 (ie., yearid = '2000'). Return both the teamid, and the number of hits. Note you can use ORDER BY column and LIMIT 1.
7. The teamid in the year 2000 (i.e., yearid = '2000')  with the highest average batting average. Return the teamid and the average. To prevent divsion by 0, limit at-bats > 0.
8. The number of all stars the giants (teamid = 'SFN') had in 2000.
9. The yearid which the giants had the most all stars.
10. The average salary in year 2000.
11. The number of positions (e.g., catchers, pitchers) that have average salaries greather than 2000000 in yearid 2000. You will need to join fielding with salaries. Also consider using a HAVING clause.
12. The number of errors Barry Bonds had in 2000. 
13. The average salary of all stars in 2000.
14. The average salary of non-all stars in 2000.


# Step 7 - Timing Plots [20 points]

Read about timeit function call at https://docs.python.org/2/library/timeit.html

Write a function that run all your queries 10 times and produces a box plot per query. Read about https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.boxplot.html

Also look at the weather box plot example in traffic example notebook in this repository


# Step 8 - Bonus [25 points]

check if you can modify your query functions and show that you can improve the time of execution by using the plots from step 7 and comparing different versions of the functions for step 6. It is required that all different versions of query functions return the correct answer. Note that you already know the correct answer from previous assignment.


