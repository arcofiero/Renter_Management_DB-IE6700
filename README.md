# Renter_Management_DB - IE6700
## Renter Management Database based on MySQL, MongoDB(NoSQL), and Python visualizations. 

IE 6700


Team Information:
Archit Raj, Sai Vinay Teja Jakku

Business Problem Definition & Requirements:
Real estate is an ever-changing and growing market. There is always a need to bridge the gap between the consumers and suppliers of the market. In this project, we attempt to close that gap by building a database that helps store, retrieve, and analyze information about the key commodity of the market i.e Houses. We do this by creating 4 tables with Renter, Owner, House, and Broker Company as key parameters. We draw relations between these parameters and their attributes and build a database that can give access to good-quality affordable housing and can even be used to strengthen the knowledge base for policy evaluation.
The following table contains how a user could link the relationship between the entities. A renter has to contact the broker company in order to get the data for the house and the owner. 
In the implementation following rules have to be kept in mind:
Broker companies can have access to multiple houses whereas houses can have access to only one broker company.
Owners can own multiple houses and can have access to more than one broker each.
Renters can deal with many broker companies and many broker companies can deal with many other renters. 
Houses can have access to at least one broker company, one renter, and one owner. 
Maintenance worker can work with many houses
Electricity department makes bills for all the houses 
				
		
Design: 
To create the database that meets the requirements of the problem statement, we first need to identify the required tables such as Renter, Owner, House, Broker Company. 
All the details of the house are being stored by the broker company, such as an address, size, number of rooms, etc. These details are being provided by the owner of the house. Every renter can deal with the broker company by providing them their details such as rent amount, term of the lease, along with their personal details such as email and phone so that they can be contacted back by the broker company. 




Conceptual Data Model - Enhanced Entity-Relationship (EER)
	


Conceptual Data Model - Unified Modeling Language (UML)




## Relational Model:
Primary Key: Bold    ||    Foreign Key: Underlined and Italics
Renter(R_ID, F_Name, L_Name, R_Age, R_Phone, R_Email, RentAmount, LeaseTerm, B_ID, H_ID) 
For Foreign Keys:
B_ID refers to the B_ID taken from B_ID in Broker_Company
	H_ID refers to the H_ID taken from H_ID in House

BrokerCompany(B_ID, F_Name, L_Name, B_Phone, B_Email, B_Address, R_ID, O_ID, H_ID)
For Foreign Keys:
R_ID refers to the R_ID taken from R_ID in Renter
	O_ID refers to the O_ID taken from O_ID in Owner
H_ID refers to the H_ID taken from H_ID in House

Owner(O_ID, F_Name, L_Name, Year_Since_Ownership, O_Phone, O_Email, B_ID, H_ID)
For Foreign Keys:
	O_ID refers to the O_ID taken from O_ID in Owner
H_ID refers to the H_ID taken from H_ID in House

House(H_ID, HouseNum, Area, Zipcode, YearEstablished, NumRooms, Size_sqft, O_ID, R_ID, B_ID )
For Foreign Keys:
	O_ID refers to the O_ID taken from O_ID in Owner
R_ID refers to the R_ID taken from R_ID in Renter
R_ID refers to the R_ID taken from R_ID in Renter

Electricity_Dept(E_ID, Amount, KwH, Bill_Date, Renter_R_ID)
For Foreign Keys:
	Renter_R_ID refers to the R_ID taken from R_ID in Renter

Maintenance_Dept(M_ID, M_Name(F_Name, L_Name), M_Date, Renetr_R_ID)
For Foreign Keys:
	Renter_R_ID refers to the R_ID taken from R_ID in Renter



## Implementation of Relation Model via MySQL and NoSQL:
## MySQL Implementation: 
The database was created in MySQL and the following queries were performed:


### Query1: Display renters who paid a rent amount $2500 with a Lease Term of 2 years. 
SELECT F_NAME, L_NAME, R_ID, Rent_Amount,   Lease_Term FROM Renter 
WHERE Rent_Amount > 2500 AND Lease_Term = 2;

### Query2: Display the name of renters whose house was established after 2000.
SELECT r.F_Name, r.L_Name, r.Rent_Amount
FROM Renter r 
LEFT JOIN House h
ON r.House_H_ID = h.H_ID
WHERE h.YearEstablished > 2000;

### Query3. Display the details of brokers with houses which are not occupied by the renters. 
SELECT * 
FROM Broker_Company
WHERE Broker_Company.B_ID
IN (SELECT Broker_Company_B_ID FROM House WHERE H_ID NOT IN (SELECT Renter.House_H_ID FROM RENTER));


### Query4. Display renter's details of those who pay below the average electricity bill.
SELECT r.F_Name, r.L_Name, r.R_Phone, r.R_Email, e.Amount
FROM Renter r
LEFT JOIN Electricity_Dept e
ON r.R_ID = e.Renter_R_ID
WHERE e.Amount < (SELECT AVG(Amount) FROM Electricity_Dept);

### Query5.  List top 3 brokers with the maximum number of renters.
SELECT b.B_ID, b.F_name, b.L_name,	
b.B_phone, COUNT(r.R_ID) AS No_of_Renters
FROM Broker_company b
JOIN House h
ON b.b_id = h.Broker_Company_B_iD
JOIN Renter r
ON r.House_H_ID = h.H_ID
GROUP BY b.B_iD
ORDER BY No_of_Renters DESC
LIMIT 3;

### Query 6.  Display the houses which haven’t been renovated in a year since they were established. 
SELECT H_ID, H_num, Area
FROM House
WHERE (YearRenovated-YearEstablished) > 1;

### Query7. Display the Owner information with most no.of houses to rent
SELECT o.O_ID, o.F_name, o.L_name, o.O_phone
FROM Owner o
JOIN House h
ON o.House_H_ID = h.H_ID
GROUP BY o.O_ID
ORDER BY COUNT(h.H_ID) DESC
LIMIT 5;

### Query 8. Display the houses with at least 3 bedrooms and ready to rent
SELECT * 
FROM House h
WHERE H_ID
IN (SELECT H_ID FROM House WHERE H_ID NOT IN (SELECT Renter.House_H_ID FROM RENTER)) AND NumRooms >= 3;



## NoSQL Implementation: 
The 6 tables(Renter, House, Broker_Company, Owner, Electricity_Dept, Maintenance) were implemented in the MongoDB as 6 different collections. 
Then, later on, the team worked on certain NoSQL’s MongoDB following queries:


### Query1: Find Renters who are aged above 30 and have a lease term of 2 years. 
db.Renter.find({ $and:[{"Lease_Term":"2"},{"R_Age":{$gt:30}}]}).pretty()


### Query2: List the total Amount of Electricity bill, KwH sorted by Renter_ID. 
db.Electricity_Dept.aggregate([
   {$match: {}},
   {$group: {_id: "$Renter_R_ID", total_Amount: {$sum: "$Amount"}, total_KwH: {$sum: "$KwH"}}},
   {$sort: {_id: 1, total_Amount:1}} ] )


### Query3: Write the query for the number of houses that have a number of rooms greater than 4 or at least Size of the house more than 2000 sq ft.  
db.House.find({ $or:[{"NumRooms":{$gt:4}}, {"Size_sqft":{$gt:2000}}]}).pretty()



7. Database Visualization through Python:
The database is accessed using Python’s PyMongo library to perform visualization of analyzed data as shown below. The connection of the MongoDB database to Python is done using PyMongo and MongoClient, followed by executing the collections to the dataframes using Pandas library and using Matplotlib and Plotly to plot the graphs for the analytics.
Graph 1: Rent Amount per Renters 
Graph 2: Renter Age Distribution Pie: 
Graph 3: Proportion of Rent Amount per Renter vs Age:
Graph 4: Area of the House vs Size per Sqft:
Graph 5: Renter Age vs Amount of Electricity Paid:

## 8. Conclusion: Outcomes and shortcomings of the Projects

1. Project Outcomes: 
We have successfully implemented our conceptual model in a SQL database and converted that to a NoSQL database.
We have demonstrated database access within the application written in Python and performed and visualized useful analytics. 
 

2. Shortcomings:
Having implemented multiple layers of database languages, MySQL, NoSQL, and then using Python libraries for getting the insights would keep it highly unstable for the long run.
Focusing more on the MongoDB platform would help the organization in gaining proper control of its whereabouts, by utilizing a cloud database setup.
For key insights, the company could shift more towards BI tools like Tableau, PowerBI, or Figma. 




