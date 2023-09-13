
# The Road Warrior System

# Vision

A new startup wants to build the next generation online trip management __dashboard__ to allow __travelers__ to see all of their __existing reservations organized by trip__ either __online (web)__ or through their __mobile device__

# Business Requirements

### Short-Term Goals (0-2 years)

- *__Develop MVP (Minimum Viable Product)__* Within the first year, create and launch a basic version of the travel dashboard that offers essential features, such as itinerary planning, real-time flight tracking, and local recommendations.

- *__Feedback and Iteration__* Gather user feedback and data to continuously improve the dashboard's user experience and functionality. Implement regular updates and fixes based on this feedback

### Long-Term Goals (3-5 years and beyond)

- *__Global Expansion__* extend the travel dashboard's reach to cover a wide range of international destinations, making it a comprehensive resource for travelers worldwide

- *__Market Leader Status__* establish the travel dashboard as a market leader in terms of user base and quality of service, becoming the go-to platform for travelers looking to plan their trips.

# Requirements

- Poll email looking for travel-related email
Filter and whitelist certain emails
- The system must interface with the agency’s existing airline, hotel, and car rental interface system to update travel details (delays, cancellations, updates, gate changes, etc.). Updates must be in the app within 5 minutes of an update (better than the competition)
- Customers should be able to add, update, or delete existing reservations manually as well
- Items in the dashboard should be able to be grouped by trip, and once the trip is complete, the items should automatically be removed from the dashboard.
- Users should also be able to share their trip information by interfacing with standard social media sites or allowing targeted people to view your trip.
- Richest user interface possible across all deployment platforms
- Provide end-of-year summary reports for users with a wide range of metrics about their travel usage
- Road Warrior gathers analytical data from users trips for various purposes - travel trends, locations, airline and hotel vendor preferences, cancellation and update frequency, and so on.
- Users must be able to access the system at all times (max 5 minutes per month of unplanned downtime)
- Travel updates must be presented in the app within 5 minutes of generation by the source
- Response time from web (800ms) and mobile (First-contentful paint of under 1.4 sec)

# The Architecture

We made the decision to use solution based on [Google Cloud](https://console.cloud.google.com/). The solution is based on the three microservices which is running in Google Cloud Run. The microservices care on the main functions our systems. Because we want to save the cost of our solution we have decided to save the date in Big Query database from Google. On the above of this db is running Vertex AI, which si responsible for standardization of all datas in Big Query. For the frontend we use the Looker from googlein order to save cost for development of Dashboard.

For the regularly polling of agentur and email, we use the Cllou Scheduler. 

![Alt text](./img/Infrastruktur.jpg)

Color legend for picture:
![Alt text](./img/Legend.jpg)

## dashboard application

The dashboard application listen the Rest api call (orange arrow). If the call is registered this microservice take the matchin data from BigQuery (1-take data). and send it back

The another service is to share the data on social nets. It coul be also registerde via Rest API (orange arrow). If the request is registerd the microservice take the data from BigQuery (1-take data) and hand over the data to *social communication* service (arrow: 2-send to social).

## social application

This microservice can be call from service dashboard (via REST API) or via Website. If the request is produce from website, the request registerd in social microservice (3-prepare data). If the request is produce via REST API the request is send from dashboard application (2-send to social).

Both producer are processing in this microservice and take the data corresponding data from BigQuery (4-pull data for sharing). This data are modifide for sharing and publish in Cloud storage (5-save in storage).

## pool email application

Pool email application is activated regulary from Cloud Scheduler CRON (6-trigged pool email). It could be for example for every 1 minutes set up. In this case the pool email application pull one fifth of users email. This way we reach that each email will be trigged with period of 5 minutes. The arrow (7-poll email) check the email and get back the relevant emails.

If the data from email are filter the data are moved to Dataflow, which will create the unified dataflow. (8-to dataflow)

## data application

This microservice has the similar function as pool email application. In this case the microservice is activeted via (9-trigged poll agentur) Cloud Scheduler CRON but with 3 minutes period. This service that ask all connected agentur API for the news. At the end of this process are the date take over to Dataflow (10-to dataflow).

## Dataflow

Service from google is responsible for save the unified data to BigQuery (11 - save data to BigQuery).

## Website

As we said, we would like to save the cost of our project. We don't want to develop any frontend site. We made the decision to use Looker. In tool we build dashboard on the base of modules from google. User will be comunicated via purple arrow. On the request from user will be data saved to BigQuery (12 - save data). 

## Big Query

On top of big query database is running VertexAI, which regulary modified the data in db.