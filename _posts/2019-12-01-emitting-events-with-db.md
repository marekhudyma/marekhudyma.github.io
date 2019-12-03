




# Asynchronous communication and two phase commit example 

# Synchronous HTTP communication 
In most cases you need to response synchronousy, for example frontend applications need to display something immediately. This communication provide many problems.
Your HTTP requests can be unsuccessful because of many reasons: 
* Http timeout - in this case we are not sure what happened: maybe server was not able to process request or server processed the request but we didn't receive response. 
* Server internal error - server may fail because of our request (and retry of the request will not help) or because temporary problems of service. 

<figure>
  <img src="/assets/2019-11-01-microservices-integration-best-practices/monolith-vs-microservices.png" alt="Monolith vs microservices"> 
  <figcaption>Monolith vs microservices</figcaption>
</figure>