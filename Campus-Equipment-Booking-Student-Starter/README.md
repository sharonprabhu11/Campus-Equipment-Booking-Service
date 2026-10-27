# Campus Equipment Booking Service

A Java 17 and Spring Boot application for managing campus equipment bookings, inventory availability, and booking cancellation. This project demonstrates Git version control, Maven build and testing, Docker containerization, and Kubernetes deployment using Minikube.

## 1. Prerequisites

- Java 17
- Apache Maven
- Git
- Docker Desktop
- Minikube
- kubectl

## 2. Build and Test

Run the following commands from the project root, where `pom.xml` is located.

Run the unit tests:

```powershell
mvn clean test
```

Package the application:

```powershell
mvn clean package
```

The generated JAR is expected at:

```text
target/campus-equipment-booking-1.0.0.jar
```

Verify the generated artifact:

```powershell
Get-ChildItem .\target\*.jar
```

## 3. Docker Build and Run

Build the application image:

```powershell
docker build -t campus-equipment-booking:set-e .
```

Verify the image:

```powershell
docker images campus-equipment-booking
```

Run the container. Port `8082` is used on the host to avoid conflicts with other services:

```powershell
docker run -d --name campus-booking -p 8082:8080 campus-equipment-booking:set-e
```

Verify the container:

```powershell
docker ps
```

Access the health endpoint:

```text
http://localhost:8082/actuator/health
```

Test it from PowerShell:

```powershell
curl.exe -i http://localhost:8082/actuator/health
```

A successful health check should return HTTP 200 with a response containing `"status":"UP"`.

## 4. Kubernetes Deployment with Minikube

Start Minikube using Docker:

```powershell
minikube start --driver=docker
```

Make the application image available to Minikube:

```powershell
minikube image load campus-equipment-booking:set-e
```

Apply the Deployment and Service manifests:

```powershell
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

Wait for the Deployment to become ready:

```powershell
kubectl rollout status deployment/campus-equipment-booking --timeout=120s
```

Inspect Kubernetes resources:

```powershell
kubectl get deployments
kubectl get pods -o wide
kubectl get services
```

### Access the health endpoint through the Service

In a separate terminal, run:

```powershell
kubectl port-forward service/campus-equipment-booking 8083:8080
```

Keep the port-forward process running. In another terminal, execute:

```powershell
curl.exe -i http://localhost:8083/actuator/health
```

The expected result is HTTP 200 with application status `UP`.

### Scale to two replicas

```powershell
kubectl scale deployment campus-equipment-booking --replicas=2
kubectl rollout status deployment/campus-equipment-booking --timeout=120s
```

Verify the replica count:

```powershell
kubectl get deployments
kubectl get pods -o wide
```

The Deployment should show two ready replicas, and two application Pods should be running.

## 5. Assigned Change Request: Booking Validation

The change request addresses incorrect inventory-capacity validation and the acceptance of invalid booking quantities.

### Inventory-capacity defect

The original capacity check used an incorrect boundary:

```java
quantity > availableQuantity + 1
```

This allowed a booking to exceed the available inventory by one unit.

The check was corrected to compare the requested quantity directly against the available quantity:

```java
quantity > availableQuantity
```

### Positive-quantity validation

Booking quantities must be greater than zero. Zero and negative quantities are rejected with an `IllegalArgumentException`, which the API maps to HTTP 400.

Validation occurs before inventory is decremented or a booking is recorded.

### Test coverage

The tests cover:

- Successful booking with a positive quantity.
- Rejection of zero quantity without changing inventory.
- Rejection of negative quantity without changing inventory.
- Acceptance of a booking for exactly
