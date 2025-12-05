# Practical 5: Refactoring a Monolithic Web Server to Microservices

## Repository
### **Source Code**: The complete source code for this practical is available in the GitHub repository:  
#### **Repository Link**: https://github.com/DechenWangdraSherpa/web303-practical-five

## Overview

This practical involved systematically refactoring a monolithic Student Cafe application into microservices using Domain-Driven Design principles. The application was decomposed into three core services (user, menu, order) with an API gateway and Consul service discovery.

## Architecture Diagram


<pre>
┌───────────────────────────────────────────────────────────┐
│                    Client Requests                        │
└──────────────────────────┬────────────────────────────────┘
                           │
                    ┌─────▼──────┐
                    │ API Gateway│ (Port 8080)
                    │            │
                    └─────┬──────┘
                          │ Routes
       ┌──────────────────┼──────────────────┐
       │                  │                  │
  ┌────▼─────┐      ┌────▼─────┐      ┌────▼─────┐
  │ User     │      │ Menu     │      │ Order    │
  │ Service  │      │ Service  │      │ Service  │
  │ (8081)   │      │ (8082)   │      │ (8083)   │
  └────┬─────┘      └────┬─────┘      └────┬─────┘
       │                  │                  │
  ┌────▼─────┐      ┌────▼─────┐      ┌────▼─────┐
  │ User DB  │      │ Menu DB  │      │ Order DB │
  │ (5434)   │      │ (5433)   │      │ (5435)   │
  └──────────┘      └──────────┘      └──────────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                    ┌──────▼──────┐
                    │ Consul      │
                    │ (8500)      │
                    │ Service     │
                    │ Discovery   │
                    └─────────────┘
</pre>

## Service Boundaries Justification

### User Service

- Bounded Context: User management (authentication, profiles)
- Entities: User
- Why separate: User data is independent of menu/orders. Can scale independently during registration spikes.

### Menu Service

- Bounded Context: Food catalog management
- Entities: MenuItem
- Why separate: Read-mostly service with high traffic (menu browsing). Price changes don't affect existing orders.

### Order Service

- Bounded Context: Order processing
- Aggregates: Order + OrderItems
- Why separate: Complex business logic for order workflow. Requires references to users and menu items via service calls.

## Implementation Challenges and Solutions

### Challenge 1: Database Schema Separation
- Problem: Each service needed its own database schema without shared tables.
- Solution: Created separate PostgreSQL instances for each service with only relevant tables.

### Challenge 2: Inter-Service Communication
- Problem: Order service needed to validate user and menu items without direct DB access.
- Solution: Implemented HTTP calls between services with proper error handling and timeouts.

### Challenge 3: Service Discovery
- Problem: Hardcoded service URLs broke when containers restarted with new IPs.
- Solution: Integrated Consul for dynamic service registration and discovery.

### Challenge 4: Data Consistency
- Problem: Order service needs price snapshot but menu service might update prices.
- Solution: Order service stores price at order time (eventual consistency acceptable).

## Reflection
### Monolith vs Microservices Comparison

For the Student Cafe application, the monolith was simpler to develop and deploy initially. All code was in one repository, database queries were straightforward, and debugging was centralized. However, as the application scales, the monolith becomes problematic. Menu browsing (high read volume) can't scale independently from order processing. Database contention occurs when different teams want to modify user profiles and menu items simultaneously.

The microservices architecture, while more complex initially, provides clear benefits: independent scaling of menu service during peak browsing hours, separate deployment of order service logic updates, and team autonomy where different developers can work on different services without stepping on each other's code.

### Database-per-Service Trade-offs

**Pros:**
- Data encapsulation - services own their data
- Technology freedom - each service can use different DB if needed
- Independent scaling - database tuning per service requirements

**Cons:**
- Distributed transactions become challenging
- Data duplication may occur (user IDs copied to orders)
- Querying across services requires API calls instead of SQL joins
- Backup and recovery more complex

For this application, the benefits outweigh the costs because the bounded contexts are well-defined and transaction boundaries align with service boundaries.

### When NOT to Split a Monolith

Microservices should not be adopted when:
- Team size is small (< 5 developers) - coordination overhead exceeds benefits
- Application is simple with limited growth projections
- Strong transaction consistency is required across domains
- Performance requirements demand low-latency local calls
- Organization lacks DevOps maturity to manage distributed systems

The Strangler Fig Pattern (gradual refactoring) is often better than a "big bang" rewrite.

### Order Service Validation Without Direct DB Access

The order service validates user existence by making an HTTP GET request to `http://user-service:8081/users/{id}`. If the response is 200 OK, the user exists. This demonstrates the principle of "share nothing" architecture - services communicate only through published APIs, not direct database access.

### Handling Menu Service Downtime During Order Creation

If menu-service is down when creating an order:
- Order service's HTTP call to menu-service times out or returns error
- Order creation fails with "Menu item not found" error
- System maintains consistency - no orders created with invalid items
- Circuit breaker pattern could be added to fail fast during prolonged outages

To improve resilience, we could:
- Implement retries with exponential backoff
- Add circuit breaker to stop calling failing service
- Cache menu items with TTL for brief outages
- Use async validation with compensating transactions

### Caching Strategy for Performance Improvement

Three-layer caching could significantly improve performance:
1. **API Gateway Cache:** Cache GET responses for /api/menu (5-minute TTL)
2. **Service-level Cache:** Redis cache in menu-service for frequent items
3. **Database Query Cache:** PostgreSQL query cache for repeated queries

Implementation would use:
- Redis for distributed caching across service instances
- Cache-aside pattern (check cache first, then database)
- Cache invalidation on menu updates via publish/subscribe
- Stale-while-revalidate for high-traffic endpoints

## Testing Commands

### Test user service
```
curl -X POST http://localhost:8080/api/users

-H "Content-Type: application/json"
-d '{"name": "Test User", "email": "test@example.com
"}'
```

### Test menu service
```
curl -X POST http://localhost:8080/api/menu

-H "Content-Type: application/json"
-d '{"name": "Burger", "description": "Beef burger", "price": 8.99}'
```

### Test order service (requires valid user_id and menu_item_id)
```
curl -X POST http://localhost:8080/api/orders

-H "Content-Type: application/json"
-d '{"user_id": 1, "items": [{"menu_item_id": 1, "quantity": 1}]}'
```

### Check Consul health
```
curl http://localhost:8500/v1/health/service/user-service
```


## Conclusion

This practical demonstrated the incremental refactoring of a monolith to microservices, highlighting the importance of domain-driven design, service boundaries, and operational concerns like service discovery. The final architecture provides independent scalability, team autonomy, and resilience while introducing complexity that must be managed through proper DevOps practices.
