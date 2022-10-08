# Ticketing app with microservices

## Features and constraints

- Only authenticated users can list or buy a ticket.
- When a user attempts to purchase a ticket, the ticket is locked for 15 mins. During the 15
mins, no other user can buy the ticket.
- The ticket owner can only edit the price when it is not locked.

## Structure and tools/libraries
The app includes several modules, a frontend client module in the React framework to show some content to the user and some backend service modules. Backend modules include:
- An authentication module
- A ticket-creating module
- An order-creating module
- An expiration module to cancel an order after 15 mins
- A payment module that cancels orders if the payment fails or completes the order if it succeeds

These backend modules are all running express servers and connected to their own MongoDB databases. These modules share some common code and they also communicate data with each other through a NATS streaming server. The entire app is deployed and runs in Dockers containers executed in a Kubernetes cluster.

## Workflow
- After an order is created, the tickets service needs to lock the ticket by adding an orderID property to the ticket. The payment service is notified that there is a new order awaiting payment. The expiration service starts counting 15 mins to time out the order.
- If the payment is successful, the payment service notifies the order service to modify the order status to "complete".
- If the order is timed out, the expiration service notifies the order service to change the order status to "canceled". The order service then tells the tickets service to unlock the ticket, and the payment service no longer accepts payment for this canceled order.
