<!-- ## Introduction
The uber.csv file is a dataset containing information about ride-sharing trips. It has 150,000 entries and provides detailed data for each ride, which can be used to analyze various aspects of the business.

Here's a breakdown of the key information included in the file:

* Ride Details: The dataset contains essential trip information such as the Date and Time of the booking, the Pickup Location and Drop Location, the Vehicle Type used, and the Ride Distance.

* Financial and Performance Metrics: Columns like Booking Value, Avg VTAT (Vehicle Travel and Arrival Time), and Avg CTAT (Customer Travel and Arrival Time) provide insights into the financial aspects and efficiency of the trips.

* Customer and Driver Data: It includes unique **Customer ID**s and ratings for both the Driver Ratings and Customer Rating, which can be used to analyze satisfaction levels.

* Trip Status: The file tracks the outcome of each booking with the Booking Status column, along with detailed information on Cancelled Rides by Customer, Cancelled Rides by Driver, and Incomplete Rides, including the reasons for these events.

* Payment: The Payment Method column details how each ride was paid for.

The dataset contains numerous missing values, particularly in the columns related to ratings and average times. These need to be handled before performing a comprehensive analysis.

### Data Cleaning and Preprocessing
```python
import pandas as pd

# Load the dataset
df = pd.read_csv("datasets/uber.csv")

# Comine "Date" and "Time" columns and convert to datetime
df["Trip Datetime"] = pd.to_datetime(df["Date"] + " " + df["Time"])
df = df.drop(columns= ["Date","Time"])

# Clean "Booking ID" and "Customer ID" columns by removing extra double quotes
df["Booking ID"] = df["Booking ID"].str.strip('\"')
df["Customer ID"] = df["Customer ID"].str.strip('\"')

# Handle missing values 
df["Avg VTAT"] = df["Avg VTAT"].fillna(0)
df["Avg CTAT"] = df["Avg CTAT"].fillna(0)
df["Cancelled Rides by Customer"] = df["Cancelled Rides by Customer"].fillna(0)
df["Reason for cancelling by Customer"] = df["Reason for cancelling by Customer"].fillna("Customer didn't cancel the ride")
df["Cancelled Rides by Driver"] = df["Cancelled Rides by Driver"].fillna(0)
df["Driver Cancellation Reason"] = df["Driver Cancellation Reason"].fillna("Driver didn't cancel the ride")
df["Incomplete Rides"] = df["Incomplete Rides"].fillna(0)
df["Incomplete Rides Reason"] = df["Incomplete Rides Reason"].fillna("No Incomplete Ride")
df["Booking Value"] = df["Booking Value"].fillna(0)
df["Ride Distance"] = df["Ride Distance"].fillna(0)
df["Driver Ratings"] = df["Driver Ratings"].fillna(0)
df["Customer Rating"] = df["Customer Rating"].fillna(0)
df["Payment Method"] = df["Payment Method"].fillna("Customer didn't pay")

# Save the cleaned dataset
df.to_csv("datasets/uber_cleaned.csv")
print("Saved Successfully")
```
### Analysis
Based on the uber.csv dataset, here are 10 analytical questions you can solve using the pandas library:

1. What are the top 5 most popular Pickup Location and Drop Location pairs, and what is the total number of rides for each?

2. What is the average Booking Value for each Vehicle Type?

3. How many rides were Completed, Cancelled Rides by Customer, or Cancelled Rides by Driver in total?

4. What is the average Customer Rating for each Payment Method?

5. What is the total number of rides and the average Ride Distance for each day of the week?

6. Which Pickup Location has the highest number of No Driver Found rides?

7. What is the distribution of Booking Status for each Vehicle Type?

8. Find the Customer ID with the highest number of rides and their total Booking Value.

9. What is the average Avg VTAT (Vehicle Travel and Arrival Time) and Avg CTAT (Customer Travel and Arrival Time) for completed rides that have a Driver Rating of 4.5 or higher?

10. How many rides were incomplete due to a Vehicle Breakdown, and what was the average Ride Distance for those rides?

#### What are the top 5 most popular Pickup Location and Drop Location pairs, and what is the total number of rides for each?
```python
import pandas as pd
import matplotlib.pyplot as plt

# Load the cleaned data
df = pd.read_csv("datasets/uber_cleaned.csv")

# Convert the "Trip Datetime"object to datetime
df["Trip Datetime"] = pd.to_datetime(df["Trip Datetime"])

# What are the top 5 most popular Pickup Location and Drop Location pairs?
top_5_routes = df.groupby(["Pickup Location","Drop Location"]).size().nlargest(5).reset_index(name="Ride Count")
print("Top 5 most popular Pickup Location and Drop Location pairs")
print(top_5_routes)

# Plotting
# Combine Pickup and Drop Location into one label for plotting
top_5_routes["Route"] = top_5_routes["Pickup Location"] + " " + top_5_routes["Drop Location"]

# Plot
plt.barh(top_5_routes["Route"],top_5_routes["Ride Count"],color="skyblue",edgecolor="black")
plt.xlabel("Ride Count")
plt.ylabel("Route (Pickup → Drop)")
plt.title("Top 5 most popular Pickup Location and Drop Location pairs")
plt.gca().invert_yaxis() # Highest at the top
plt.grid(axis="x",linestyle="--",alpha=0.7)

# Show the plot
plt.tight_layout()
plt.show()
```
![question1](/assets/question1.png)

#### What is the average Booking Value for each Vehicle Type?
```python
import pandas as pd
import matplotlib.pyplot as plt

# Load the cleaned data
df = pd.read_csv("datasets/uber_cleaned.csv")

# What is the average Booking Value for each Vehicle Type?
avg_booking_value_by_vehicle = df.groupby("Vehicle Type")["Booking Value"].mean().reset_index()
print("Average Booking Value for each Vehicle Type")
print(avg_booking_value_by_vehicle)

# Plot
plt.bar(avg_booking_value_by_vehicle["Vehicle Type"],
        avg_booking_value_by_vehicle["Booking Value"],
        color="skyblue",
        edgecolor="black")

# Label and title
plt.xlabel("Vehicle Type")
plt.ylabel("Booking Value")
plt.title("Average Booking Value by Vehicle Type")
plt.xticks(rotation=45)
plt.grid(axis="y",linestyle="--",alpha=0.7)

# Show the plot 
plt.tight_layout()
plt.show()
```
![question2](/assets/question2.png)

#### How many rides were Completed, Cancelled Rides by Customer, or Cancelled Rides by Driver in total?
```python
import pandas as pd
import matplotlib.pyplot as plt

# Load the Cleaned data
df = pd.read_csv("datasets/uber_cleaned.csv")

# How many rides were Completed, Cancelled by Customer or Cancelled by Driver
completed_rides = df[df["Booking Status"] == "Completed"].shape[0]
customer_cancelled = df["Cancelled Rides by Customer"].sum()
driver_cancelled = df["Cancelled Rides by Driver"].sum()
print("Total number of Completed,Cancelled by Customer and Cancelled by Driver rides:")
print(f"Completed Rides: {completed_rides}")
print(f"Cancelled Rides by Customer: {int(customer_cancelled)}")
print(f"Cancelled Rides by Driver: {int(driver_cancelled)}")

# Prepare data for Plotting
categories = ["Completed Rides","Cancelled by Customers","Cancelled by Driver"]
values = [completed_rides,customer_cancelled,driver_cancelled]

# Create the bar chart
bars = plt.bar(categories,values,color=["skyblue","darkblue","mediumblue"],edgecolor="black")

# Labels and title
plt.ylabel("Number of Rides")
plt.title("Completed,Cancelled by Customer and Cancelled by Driver Rides")
plt.grid(axis="y",linestyle= "--",alpha=0.7)

# Show the plot 
plt.tight_layout()
plt.show()
```
![question3](/assets/question3.png)

#### What is the average Customer Rating for each Payment Method?
```python
import pandas as pd
import matplotlib.pyplot as plt

# Load the Cleaned data
df = pd.read_csv("datasets/uber_cleaned.csv")

# What is the Average Customer Rating of each Payment Method
avg_customer_rating_by_payment = df.groupby("Payment Method")["Customer Rating"].mean().reset_index()
print("Average Customer Rating for each Payment Method")
print(avg_customer_rating_by_payment)

# Plot
bars = plt.bar(
    avg_customer_rating_by_payment["Payment Method"],
    avg_customer_rating_by_payment["Customer Rating"],
    color="skyblue",
    edgecolor="black"
)

# Label and title
plt.xlabel("Payment Method")
plt.ylabel("Average Customer Rating")
plt.title("Average Customer Rating by Payment Method")
plt.ylim(0,5) # Ratings are usually between 0 and 5
plt.xticks(rotation=45)
plt.grid(axis="y",linestyle="--",alpha=0.7)

# Show plot
plt.tight_layout()
plt.show()
```
![question4](/assets/question4.png)

#### What is the total number of rides and the average Ride Distance for each day of the week?
```python
import pandas as pd
import matplotlib.pyplot as plt

# Load the Cleaned data
df = pd.read_csv("datasets/uber_cleaned.csv")

# Convert the "Trip Datetime" object to datetime
df["Trip Datetime"] = pd.to_datetime(df["Trip Datetime"])

# What is the total number of rides and the average ride distance for each day of the week
df["Day of Week"] = df["Trip Datetime"].dt.day_name()
rises_by_day = df.groupby("Day of Week").agg(Total_Rides=("Booking ID","count"),Average_Ride_Distance=("Ride Distance","mean")).reset_index()
rises_by_day =rises_by_day.rename(columns={
    "Total_Rides":"Total Rides",
    "Average_Ride_Distance":"Average Ride Distance"
})

# Display the Result
print("Total number of rides and Average ride distance for each day of the week")
print(rises_by_day)

# Plotting
# Ensure the days are in proper week order
days_order = ["Monday","Tuesday","Wednesday","Thursday","Friday","Saturday","Sunday"]
rises_by_day["Day of Week"] = pd.Categorical(rises_by_day["Day of Week"],categories=days_order,ordered=True)
rises_by_day = rises_by_day.sort_values("Day of Week")

# Create a Plot
fig,ax1 = plt.subplots(figsize=(10,6))

# Plot the Total Rides on the left y-axis
bars = ax1.bar(rises_by_day["Day of Week"],rises_by_day["Total Rides"],color="skyblue",label="Total Rides")
ax1.set_xlabel("Day of Week")
ax1.set_ylabel("Total Rides",color="blue")
ax1.tick_params(axis="y",labelcolor="blue")


# Create a second y-axis for Average Ride Distance
ax2 = ax1.twinx()
ax2.plot(rises_by_day["Day of Week"],rises_by_day["Average Ride Distance"],color = "red",marker = "o",label="Average Ride Distance")
ax2.set_ylabel("Average Ride Distance",color = "red")
ax2.tick_params(axis="y",labelcolor= "red")

# Title and grid
plt.title("Total Rides and Average Ride Distance of each day of the week")
ax1.grid(axis="y",linestyle="--",alpha= 0.6)

# Legends
fig.legend(loc="upper right", bbox_to_anchor=(1, 1), bbox_transform=ax1.transAxes)


# Show Plot
plt.tight_layout() 
plt.show()
```
![question5](/assets/question5.png)

#### Which Pickup Location has the highest number of No Driver Found rides?
```python
import pandas as pd

# Load the Cleaned Data
df = pd.read_csv("datasets/uber_cleaned.csv")

# Which Pickup Location has the highest number of No Driver Found rides?
no_driver_found_locations = df[df["Booking Status"] == "No Driver Found"]["Pickup Location"].value_counts().idxmax()
print("Pickup Location with the highest number of 'No Driver Found' rides:")
print(no_driver_found_locations)


```

#### What is the distribution of Booking Status for each Vehicle Type?
```python
import pandas as pd
import matplotlib.pyplot as plt

# Load the Cleaned Data
df = pd.read_csv("datasets/uber_cleaned.csv")

# What is the distribution of Booking Status for each Vehicle Type?
status_by_vehicle = pd.crosstab(df["Vehicle Type"], df["Booking Status"])
print("Distribution of Booking Status for each Vehicle Type")
print(status_by_vehicle)

# Plot
status_by_vehicle.plot(kind="bar",color=["blue","skyblue","mediumblue","darkblue","royalblue"])

# Labels and title
plt.xlabel("Vehicle Type")
plt.ylabel("Booking Status")
plt.title("Distribution of Booking Status by Vehicle Type")
plt.xticks(rotation=45)
plt.grid(axis="y",linestyle="--",alpha=0.7)
plt.legend(title="Booking Status")

# Show the Plot
plt.tight_layout()
plt.show()
```
![question7](/assets/question7.png)


#### Find the Customer ID with the highest number of rides and their total Booking Value.
```python
import pandas as pd
import matplotlib.pyplot as plt

# Load the cleaned data
df = pd.read_csv("datasets/uber_cleaned.csv")

# Customer ID with the highest number of rides and their total Booking Value
top_customer_id = df["Customer ID"].value_counts().idxmax()
total_booking_value = df[df["Customer ID"] == top_customer_id]["Booking Value"].sum()
print("Customer with the highest number of rides and their total Booking Value:")
print(f"Top Customer ID: {top_customer_id}")
print(f"Total Booking Value for this customer: {total_booking_value:.2f}")

```

#### What is the average Avg VTAT (Vehicle Travel and Arrival Time) and Avg CTAT (Customer Travel and Arrival Time) for completed rides that have a Driver Rating of 4.5 or higher?
```python
import pandas as pd
import matplotlib.pyplot as plt

# Load the Cleaned Data
df = pd.read_csv("datasets/uber_cleaned.csv")

# Average Avg VTAT and Avg CTAT for completed rides with a Driver Rating 4.5 or higher
filtered_rides = df[(df["Booking Status"] == "Completed") & (df["Driver Ratings"] >= 4.5)]
avg_vtat = filtered_rides["Avg VTAT"].mean()
avg_ctat = filtered_rides["Avg CTAT"].mean()
print("Average VTAT and CTAT for completed rides with a Driver Rating of 4.5 or higher")
print(f"Average VTAT: {avg_vtat:.2f}")
print(f"Average CTAT: {avg_ctat:.2f}")

# Plotting
metrics = ["Average VTAT","Average CTAT"]
values = [avg_vtat,avg_ctat]

# Create a bar chart
plt.figure()
bars = plt.bar(metrics,values,color=["skyblue","darkblue"],edgecolor="black")

# Labels & tittle
plt.title("Average VTAT & CTAT (Driver Rating >= 4.5, Completed Rides)")
plt.ylabel("Time (minutes)")
plt.grid(axis="y",linestyle="--",alpha=0.7)

# Display the Plot
plt.tight_layout()
plt.show()
```
![question9](/assets/question9.png)

#### How many rides were incomplete due to a Vehicle Breakdown, and what was the average Ride Distance for those rides?
```python
import pandas as pd
import matplotlib.pyplot as plt

# Load the cleaned data
df = pd.read_csv("datasets/uber_cleaned.csv")

# How many incomplete rides due to 'Vehicle Breakdown' and what was the average Ride Distance?
vehicle_breakdown_rides = df[df["Incomplete Rides Reason"] == "Vehicle Breakdown"]
count_breakdown_rides =vehicle_breakdown_rides.shape[0]
avg_distance_breakdown = vehicle_breakdown_rides["Ride Distance"].mean()

# Display the Result
print(f"Incomplete rises due to 'Vehicle Breakdown' and their average Ride Distance")
print(f"Total incomplete rises due to 'Vehicle Breakdown':{count_breakdown_rides}")
print(f"Average ride distance for these rides: {avg_distance_breakdown:.2f}")


# Plotting
metrics = ["Total Incomplete Rides","Average Ride Distance"]
values = [count_breakdown_rides,avg_distance_breakdown]

# Create a figure
plt.figure()
bars = plt.bar(metrics,values,color=["tomato","skyblue"],edgecolor="black")

# Title and styling
plt.title("Incomplete Rides Due to Vehicle Breakdown",fontsize=14)
plt.ylabel("Value")
plt.grid(axis="y",linestyle="--",alpha=0.7)

# Display the plot
plt.tight_layout()
plt.show()
```
-->










<h1 align="center">Uber Ride Demand Pricing Insights</h1>

![uber-header](/assets/header.png)