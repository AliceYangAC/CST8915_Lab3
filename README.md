# Lab 3

## Demo Video

[Youtube Link](https://youtu.be/LekPwWe7yhk)

## Reflection Questions

1. **What challenges did you encounter when configuring environment variables in the GitHub Actions workflow?**

As we had discussed in class, unlike `order-service` and `product-service,` the static frontend `store-front` application compiles into static HTML, CSS, and JavaScript files that are served to the browser on deployment. Since it is a serverless application that runs in the user's browser, it cannot dynamically read environment variables at runtime. Therefore, the environment variables must be defined in the GitHub Actions workflow file so the frontend app bakes the environment variables during build, and is able to know how to reach the backend services.

2. **How does deploying microservices on Azure Web App Service differ from running them locally?**

When we run them locally, they communicate with each other through uniquely set ports (ie. `order-service` set to port 3000) that were initially hardcoded into the app, and later externalized into environment variables in `.env` files. CORS is a dependency that allows our multiple microservices to interact with each other from different ports. When all of the microservices are built via command line, they are able to communicate with each other through their ports like `store-front` making an HTTP request to GET product lists to display from `product-service` and `order-service` publishing a new order event to their topic in RabbitMQ.

However, when we deploy `order-service`, `product-service` and `store-front` on Azure Web App Services, there are a couple key differences. For `order-service` and `product-service`, rather than externalizing the environment variables (like ports) into a local `.env` file, we configure them directly in the App Service Settings interface. If we wanted to, we could also then connect the apps to other Azure networking resources like Azure VNet, App Gateway, or Front Door; locally, we are limited to localhost or LAN connections. Furthermore, since `store-front` is deployed on Static Web App Service, it must bake its environment variables that determine the backend services' connections during build. (as described in question 1) The two URLs that `store-front` connects to are no longer the IP addresses and port numbers for their respective backend services that are all running on a single machine, but instead the Azure-generated public URLs for the separate App Services hosting the two backend services. In addition, App Services supports GitHub Actions to automate CI/CD pipelines for building in deploying instead of the manual builds done locally.

Finally, the connection string that allows `order-service` to communicate with RabbitMQ is the same syntax, but with a different IP address than when ran locally:

`RABBITMQ_CONNECTION_STRING=amqp://<username>:<password>@<IP address>:5672/`

Since we are hosting RabbitMQ remotely on a VM instead of on our local machine, we have to set the IP address to the public IP of the VM instead of localhost.

1. **Why is it important to use environment variables for configurations in a cloud environment?**

Environment variables are important in a cloud environment to support microservice isolation, flexibility and overall security. Like the 'Config' factor in the 12 Factors architectural design system dictates, you can flexibly change environment variables between various environments (staging, production, developer, etc) without touching the code at all, thus allowing you to configure freely without having to rebuild the app or affecting any other service in the system. The externalized configurations also supports automated CI/CD pipelines as a result of the same flexibility to configure variables separate from the deployed codebase. Furthermore, environment variables configured in Azure UIs work consistently across all Azure resources that communicate with each other in your ecosystem. In addition, environment variables enforce security by keeping sensitive information like tokens or keys out of source code and repo history, reduce risk of any breaches.

## Links

[Order Service repo](https://github.com/AliceYangAC/order-service)

[Product Service repo (original Rust)](https://github.com/AliceYangAC/product-service)

[Product Service repo (refactored into Python)](https://github.com/AliceYangAC/product-service-python-refactored)

[Store Front repo](https://github.com/AliceYangAC/store-front)

## Notes

- I learned an important lesson to not constantly reboot the App Services whenever I make changes; it is not required, and I hit the 'Quota Exceeded' status on the Free F1 tier as a result.
- Worst case if I am dealing with persistent errors, deleting the lab resource group and restarting is not a bad idea; sometimes Azure is just funky.
