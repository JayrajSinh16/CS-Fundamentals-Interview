# Microservices Architecture

Microservices architecture is a design pattern where applications are structured as a collection of loosely coupled, independently deployable services. Each service is responsible for a specific business capability and communicates through well-defined APIs.

## 🏗️ Microservices Architecture Overview

```mermaid
graph TB
    A[API Gateway] --> B[User Service]
    A --> C[Order Service] 
    A --> D[Payment Service]
    A --> E[Notification Service]
    A --> F[Inventory Service]
    
    B --> G[(User DB)]
    C --> H[(Order DB)]
    D --> I[(Payment DB)]
    E --> J[Message Queue]
    F --> K[(Inventory DB)]
    
    L[Service Discovery] --> A
    L --> B
    L --> C
    L --> D
    L --> E
    L --> F
    
    M[Config Server] --> B
    M --> C
    M --> D
    M --> E
    M --> F
    
    style A fill:#ffcccc
    style L fill:#ccffcc
    style M fill:#ffffcc
```

## 🔍 Core Principles

### 1. Single Responsibility Principle
Each microservice should have one reason to change and own a specific business domain.

```python
from abc import ABC, abstractmethod
from typing import Dict, List, Optional
import uuid
import time

# User Service - Manages user lifecycle
class UserService:
    def __init__(self, user_repository):
        self.user_repository = user_repository
        self.event_publisher = EventPublisher()
    
    def create_user(self, user_data: Dict) -> Dict:
        """Create new user"""
        user = {
            'id': str(uuid.uuid4()),
            'email': user_data['email'],
            'username': user_data['username'],
            'created_at': time.time(),
            'status': 'active'
        }
        
        # Save to database
        self.user_repository.save(user)
        
        # Publish event for other services
        self.event_publisher.publish('user.created', {
            'user_id': user['id'],
            'email': user['email'],
            'timestamp': user['created_at']
        })
        
        return user
    
    def get_user(self, user_id: str) -> Optional[Dict]:
        """Get user by ID"""
        return self.user_repository.find_by_id(user_id)
    
    def update_user(self, user_id: str, updates: Dict) -> bool:
        """Update user information"""
        user = self.user_repository.find_by_id(user_id)
        if not user:
            return False
        
        user.update(updates)
        user['updated_at'] = time.time()
        
        self.user_repository.save(user)
        
        # Publish update event
        self.event_publisher.publish('user.updated', {
            'user_id': user_id,
            'updates': updates,
            'timestamp': time.time()
        })
        
        return True
    
    def delete_user(self, user_id: str) -> bool:
        """Soft delete user"""
        user = self.user_repository.find_by_id(user_id)
        if not user:
            return False
        
        user['status'] = 'deleted'
        user['deleted_at'] = time.time()
        
        self.user_repository.save(user)
        
        # Publish deletion event
        self.event_publisher.publish('user.deleted', {
            'user_id': user_id,
            'timestamp': time.time()
        })
        
        return True

# Order Service - Manages order lifecycle
class OrderService:
    def __init__(self, order_repository, user_service_client, inventory_service_client):
        self.order_repository = order_repository
        self.user_service = user_service_client
        self.inventory_service = inventory_service_client
        self.event_publisher = EventPublisher()
    
    def create_order(self, order_data: Dict) -> Dict:
        """Create new order with validation"""
        # Validate user exists
        user = self.user_service.get_user(order_data['user_id'])
        if not user:
            raise ValueError("User not found")
        
        # Validate inventory
        for item in order_data['items']:
            available = self.inventory_service.check_availability(
                item['product_id'], item['quantity']
            )
            if not available:
                raise ValueError(f"Product {item['product_id']} not available")
        
        # Create order
        order = {
            'id': str(uuid.uuid4()),
            'user_id': order_data['user_id'],
            'items': order_data['items'],
            'total_amount': self._calculate_total(order_data['items']),
            'status': 'pending',
            'created_at': time.time()
        }
        
        self.order_repository.save(order)
        
        # Reserve inventory
        for item in order['items']:
            self.inventory_service.reserve_item(
                item['product_id'], item['quantity'], order['id']
            )
        
        # Publish order created event
        self.event_publisher.publish('order.created', {
            'order_id': order['id'],
            'user_id': order['user_id'],
            'total_amount': order['total_amount'],
            'timestamp': order['created_at']
        })
        
        return order
    
    def _calculate_total(self, items: List[Dict]) -> float:
        """Calculate order total"""
        total = 0
        for item in items:
            # In practice, get price from product service
            total += item['price'] * item['quantity']
        return total
    
    def get_order(self, order_id: str) -> Optional[Dict]:
        """Get order by ID"""
        return self.order_repository.find_by_id(order_id)
    
    def update_order_status(self, order_id: str, status: str) -> bool:
        """Update order status"""
        order = self.order_repository.find_by_id(order_id)
        if not order:
            return False
        
        old_status = order['status']
        order['status'] = status
        order['updated_at'] = time.time()
        
        self.order_repository.save(order)
        
        # Publish status change event
        self.event_publisher.publish('order.status_changed', {
            'order_id': order_id,
            'old_status': old_status,
            'new_status': status,
            'timestamp': time.time()
        })
        
        return True
```

### 2. Decentralized Data Management
Each service manages its own data and database.

```python
# Database per service pattern
class ServiceDatabase:
    def __init__(self, service_name: str, connection_string: str):
        self.service_name = service_name
        self.connection = self._create_connection(connection_string)
        self._setup_tables()
    
    def _create_connection(self, connection_string: str):
        # Database connection logic
        pass
    
    def _setup_tables(self):
        # Service-specific table creation
        pass

class UserRepository:
    def __init__(self, database: ServiceDatabase):
        self.db = database
    
    def save(self, user: Dict) -> bool:
        # Save user to user service database
        pass
    
    def find_by_id(self, user_id: str) -> Optional[Dict]:
        # Find user in user service database
        pass
    
    def find_by_email(self, email: str) -> Optional[Dict]:
        # Find user by email
        pass

class OrderRepository:
    def __init__(self, database: ServiceDatabase):
        self.db = database
    
    def save(self, order: Dict) -> bool:
        # Save order to order service database
        pass
    
    def find_by_id(self, order_id: str) -> Optional[Dict]:
        # Find order in order service database
        pass
    
    def find_by_user_id(self, user_id: str) -> List[Dict]:
        # Find orders for a user
        pass
```

## 🔗 Service Communication Patterns

### 1. Synchronous Communication (HTTP/REST)

```python
import requests
import json
from typing import Optional
import time

class ServiceClient:
    """Base class for service-to-service communication"""
    
    def __init__(self, base_url: str, timeout: int = 30, max_retries: int = 3):
        self.base_url = base_url.rstrip('/')
        self.timeout = timeout
        self.max_retries = max_retries
        self.session = requests.Session()
    
    def _make_request(self, method: str, endpoint: str, 
                     data: Dict = None, params: Dict = None) -> Optional[Dict]:
        """Make HTTP request with retry logic"""
        url = f"{self.base_url}{endpoint}"
        
        for attempt in range(self.max_retries + 1):
            try:
                response = self.session.request(
                    method=method,
                    url=url,
                    json=data,
                    params=params,
                    timeout=self.timeout
                )
                
                if response.status_code == 200:
                    return response.json()
                elif response.status_code == 404:
                    return None
                else:
                    response.raise_for_status()
                    
            except requests.RequestException as e:
                if attempt == self.max_retries:
                    raise ServiceCommunicationError(f"Failed to call {url}: {e}")
                
                # Exponential backoff
                wait_time = (2 ** attempt) * 0.1
                time.sleep(wait_time)
        
        return None
    
    def get(self, endpoint: str, params: Dict = None) -> Optional[Dict]:
        return self._make_request('GET', endpoint, params=params)
    
    def post(self, endpoint: str, data: Dict) -> Optional[Dict]:
        return self._make_request('POST', endpoint, data=data)
    
    def put(self, endpoint: str, data: Dict) -> Optional[Dict]:
        return self._make_request('PUT', endpoint, data=data)
    
    def delete(self, endpoint: str) -> bool:
        result = self._make_request('DELETE', endpoint)
        return result is not None

class UserServiceClient(ServiceClient):
    """Client for User Service"""
    
    def get_user(self, user_id: str) -> Optional[Dict]:
        return self.get(f'/users/{user_id}')
    
    def create_user(self, user_data: Dict) -> Optional[Dict]:
        return self.post('/users', user_data)
    
    def update_user(self, user_id: str, updates: Dict) -> bool:
        result = self.put(f'/users/{user_id}', updates)
        return result is not None

class InventoryServiceClient(ServiceClient):
    """Client for Inventory Service"""
    
    def check_availability(self, product_id: str, quantity: int) -> bool:
        result = self.get(f'/inventory/{product_id}/availability', 
                         params={'quantity': quantity})
        return result.get('available', False) if result else False
    
    def reserve_item(self, product_id: str, quantity: int, order_id: str) -> bool:
        result = self.post(f'/inventory/{product_id}/reserve', {
            'quantity': quantity,
            'order_id': order_id
        })
        return result is not None
    
    def release_reservation(self, product_id: str, order_id: str) -> bool:
        return self.delete(f'/inventory/{product_id}/reservations/{order_id}')

class ServiceCommunicationError(Exception):
    pass
```

### 2. Asynchronous Communication (Event-Driven)

```python
import json
import threading
from typing import Callable, Dict, List
from dataclasses import dataclass
from queue import Queue

@dataclass
class Event:
    event_type: str
    payload: Dict
    timestamp: float
    correlation_id: str = None
    source_service: str = None

class EventPublisher:
    """Publishes events to message broker"""
    
    def __init__(self, message_broker):
        self.message_broker = message_broker
    
    def publish(self, event_type: str, payload: Dict, 
               correlation_id: str = None, source_service: str = None):
        """Publish event to message broker"""
        event = Event(
            event_type=event_type,
            payload=payload,
            timestamp=time.time(),
            correlation_id=correlation_id,
            source_service=source_service
        )
        
        self.message_broker.publish(event_type, event)

class EventSubscriber:
    """Subscribes to events from message broker"""
    
    def __init__(self, message_broker, service_name: str):
        self.message_broker = message_broker
        self.service_name = service_name
        self.handlers = {}  # event_type -> handler function
        self.running = False
    
    def subscribe(self, event_type: str, handler: Callable[[Event], None]):
        """Subscribe to specific event type"""
        self.handlers[event_type] = handler
        self.message_broker.subscribe(event_type, self._handle_event)
    
    def _handle_event(self, event: Event):
        """Handle incoming event"""
        handler = self.handlers.get(event.event_type)
        if handler:
            try:
                handler(event)
            except Exception as e:
                print(f"Error handling event {event.event_type}: {e}")
    
    def start(self):
        """Start listening for events"""
        self.running = True
        self.message_broker.start_consuming()
    
    def stop(self):
        """Stop listening for events"""
        self.running = False
        self.message_broker.stop_consuming()

# Example event handlers
class NotificationService:
    def __init__(self, event_subscriber: EventSubscriber):
        self.event_subscriber = event_subscriber
        self._setup_event_handlers()
    
    def _setup_event_handlers(self):
        """Setup event handlers"""
        self.event_subscriber.subscribe('user.created', self._handle_user_created)
        self.event_subscriber.subscribe('order.created', self._handle_order_created)
        self.event_subscriber.subscribe('payment.completed', self._handle_payment_completed)
    
    def _handle_user_created(self, event: Event):
        """Send welcome email when user is created"""
        user_data = event.payload
        self._send_welcome_email(user_data['email'], user_data['user_id'])
    
    def _handle_order_created(self, event: Event):
        """Send order confirmation when order is created"""
        order_data = event.payload
        self._send_order_confirmation(order_data['user_id'], order_data['order_id'])
    
    def _handle_payment_completed(self, event: Event):
        """Send payment receipt when payment is completed"""
        payment_data = event.payload
        self._send_payment_receipt(payment_data['user_id'], payment_data['order_id'])
    
    def _send_welcome_email(self, email: str, user_id: str):
        # Email sending logic
        print(f"Sending welcome email to {email}")
    
    def _send_order_confirmation(self, user_id: str, order_id: str):
        # Order confirmation logic
        print(f"Sending order confirmation for {order_id} to user {user_id}")
    
    def _send_payment_receipt(self, user_id: str, order_id: str):
        # Payment receipt logic
        print(f"Sending payment receipt for {order_id} to user {user_id}")
```

## 🛡️ Service Resilience Patterns

### 1. Circuit Breaker Pattern

```python
import time
import threading
from enum import Enum
from typing import Callable, Any, Optional

class CircuitState(Enum):
    CLOSED = "closed"
    OPEN = "open"
    HALF_OPEN = "half_open"

class CircuitBreaker:
    """Circuit breaker pattern implementation"""
    
    def __init__(self, failure_threshold: int = 5, timeout: int = 60, 
                 expected_exception: type = Exception):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.expected_exception = expected_exception
        
        self.failure_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED
        self.success_count = 0
        
        self._lock = threading.Lock()
    
    def call(self, func: Callable, *args, **kwargs) -> Any:
        """Execute function with circuit breaker protection"""
        with self._lock:
            if self.state == CircuitState.OPEN:
                if self._should_attempt_reset():
                    self.state = CircuitState.HALF_OPEN
                    self.success_count = 0
                else:
                    raise CircuitBreakerOpenError("Circuit breaker is open")
            
            try:
                result = func(*args, **kwargs)
                self._on_success()
                return result
                
            except self.expected_exception as e:
                self._on_failure()
                raise e
    
    def _should_attempt_reset(self) -> bool:
        """Check if enough time has passed to attempt reset"""
        return (self.last_failure_time and 
                time.time() - self.last_failure_time >= self.timeout)
    
    def _on_success(self):
        """Handle successful call"""
        if self.state == CircuitState.HALF_OPEN:
            self.success_count += 1
            if self.success_count >= 3:  # Require 3 successes to close
                self.state = CircuitState.CLOSED
                self.failure_count = 0
        elif self.state == CircuitState.CLOSED:
            self.failure_count = 0
    
    def _on_failure(self):
        """Handle failed call"""
        self.failure_count += 1
        self.last_failure_time = time.time()
        
        if self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN
    
    def get_state(self) -> Dict:
        """Get current circuit breaker state"""
        return {
            'state': self.state.value,
            'failure_count': self.failure_count,
            'last_failure_time': self.last_failure_time,
            'failure_threshold': self.failure_threshold
        }

class CircuitBreakerOpenError(Exception):
    pass

# Usage with service clients
class ResilientServiceClient(ServiceClient):
    """Service client with circuit breaker"""
    
    def __init__(self, base_url: str, **kwargs):
        super().__init__(base_url, **kwargs)
        self.circuit_breaker = CircuitBreaker(
            failure_threshold=5,
            timeout=60,
            expected_exception=ServiceCommunicationError
        )
    
    def _make_request(self, method: str, endpoint: str, 
                     data: Dict = None, params: Dict = None) -> Optional[Dict]:
        """Make request with circuit breaker protection"""
        return self.circuit_breaker.call(
            super()._make_request, method, endpoint, data, params
        )
```

### 2. Retry Pattern with Exponential Backoff

```python
import random
import time
from typing import Callable, Any, Type, Tuple

class RetryPolicy:
    """Configurable retry policy"""
    
    def __init__(self, max_attempts: int = 3, base_delay: float = 1.0,
                 max_delay: float = 60.0, exponential_base: float = 2.0,
                 jitter: bool = True, retryable_exceptions: Tuple[Type[Exception], ...] = None):
        self.max_attempts = max_attempts
        self.base_delay = base_delay
        self.max_delay = max_delay
        self.exponential_base = exponential_base
        self.jitter = jitter
        self.retryable_exceptions = retryable_exceptions or (Exception,)
    
    def execute(self, func: Callable, *args, **kwargs) -> Any:
        """Execute function with retry policy"""
        last_exception = None
        
        for attempt in range(self.max_attempts):
            try:
                return func(*args, **kwargs)
                
            except Exception as e:
                last_exception = e
                
                # Check if exception is retryable
                if not isinstance(e, self.retryable_exceptions):
                    raise e
                
                # Don't delay on last attempt
                if attempt == self.max_attempts - 1:
                    break
                
                # Calculate delay
                delay = min(
                    self.base_delay * (self.exponential_base ** attempt),
                    self.max_delay
                )
                
                # Add jitter to prevent thundering herd
                if self.jitter:
                    delay *= (0.5 + random.random() * 0.5)
                
                time.sleep(delay)
        
        # All attempts failed
        raise last_exception

# Usage
class PaymentService:
    def __init__(self, payment_gateway_client):
        self.payment_gateway = payment_gateway_client
        self.retry_policy = RetryPolicy(
            max_attempts=3,
            base_delay=1.0,
            retryable_exceptions=(PaymentGatewayError, requests.RequestException)
        )
    
    def process_payment(self, payment_data: Dict) -> Dict:
        """Process payment with retry logic"""
        return self.retry_policy.execute(
            self._make_payment_request, payment_data
        )
    
    def _make_payment_request(self, payment_data: Dict) -> Dict:
        """Make actual payment request"""
        return self.payment_gateway.charge(payment_data)

class PaymentGatewayError(Exception):
    pass
```

### 3. Bulkhead Pattern

```python
import threading
from concurrent.futures import ThreadPoolExecutor, TimeoutError
from typing import Callable, Any

class BulkheadExecutor:
    """Isolate different types of requests using separate thread pools"""
    
    def __init__(self):
        self.executors = {}
        self.default_pool_size = 10
    
    def create_bulkhead(self, name: str, max_workers: int = None):
        """Create isolated thread pool for specific operations"""
        max_workers = max_workers or self.default_pool_size
        self.executors[name] = ThreadPoolExecutor(
            max_workers=max_workers,
            thread_name_prefix=f"bulkhead-{name}"
        )
    
    def execute_in_bulkhead(self, bulkhead_name: str, func: Callable, 
                           *args, timeout: int = 30, **kwargs) -> Any:
        """Execute function in specific bulkhead"""
        if bulkhead_name not in self.executors:
            raise ValueError(f"Bulkhead {bulkhead_name} not found")
        
        executor = self.executors[bulkhead_name]
        future = executor.submit(func, *args, **kwargs)
        
        try:
            return future.result(timeout=timeout)
        except TimeoutError:
            future.cancel()
            raise BulkheadTimeoutError(f"Operation timed out in bulkhead {bulkhead_name}")
    
    def shutdown_all(self):
        """Shutdown all bulkheads"""
        for executor in self.executors.values():
            executor.shutdown(wait=True)

class BulkheadTimeoutError(Exception):
    pass

# Usage example
class UserService:
    def __init__(self):
        self.bulkhead_executor = BulkheadExecutor()
        
        # Create separate bulkheads for different operations
        self.bulkhead_executor.create_bulkhead('user_queries', max_workers=20)
        self.bulkhead_executor.create_bulkhead('user_updates', max_workers=5)
        self.bulkhead_executor.create_bulkhead('user_reports', max_workers=3)
    
    def get_user(self, user_id: str) -> Dict:
        """Get user using query bulkhead"""
        return self.bulkhead_executor.execute_in_bulkhead(
            'user_queries', self._fetch_user_from_db, user_id
        )
    
    def update_user(self, user_id: str, updates: Dict) -> bool:
        """Update user using update bulkhead"""
        return self.bulkhead_executor.execute_in_bulkhead(
            'user_updates', self._update_user_in_db, user_id, updates
        )
    
    def generate_user_report(self, user_id: str) -> Dict:
        """Generate report using report bulkhead"""
        return self.bulkhead_executor.execute_in_bulkhead(
            'user_reports', self._generate_report, user_id, timeout=60
        )
    
    def _fetch_user_from_db(self, user_id: str) -> Dict:
        # Database query logic
        pass
    
    def _update_user_in_db(self, user_id: str, updates: Dict) -> bool:
        # Database update logic
        pass
    
    def _generate_report(self, user_id: str) -> Dict:
        # Report generation logic (potentially long-running)
        pass
```

## 🔍 Service Discovery

### Service Registry Implementation

```python
import time
import threading
from typing import Dict, List, Optional
from dataclasses import dataclass, field

@dataclass
class ServiceInstance:
    service_name: str
    instance_id: str
    host: str
    port: int
    health_check_url: str = None
    metadata: Dict = field(default_factory=dict)
    registered_at: float = field(default_factory=time.time)
    last_heartbeat: float = field(default_factory=time.time)
    status: str = "UP"

class ServiceRegistry:
    """Centralized service registry"""
    
    def __init__(self, heartbeat_interval: int = 30, cleanup_interval: int = 60):
        self.services = {}  # service_name -> {instance_id -> ServiceInstance}
        self.heartbeat_interval = heartbeat_interval
        self.cleanup_interval = cleanup_interval
        self._lock = threading.RLock()
        
        # Background thread for cleanup
        self.cleanup_thread = threading.Thread(target=self._cleanup_loop, daemon=True)
        self.running = True
        self.cleanup_thread.start()
    
    def register_service(self, service_instance: ServiceInstance) -> bool:
        """Register service instance"""
        with self._lock:
            service_name = service_instance.service_name
            instance_id = service_instance.instance_id
            
            if service_name not in self.services:
                self.services[service_name] = {}
            
            self.services[service_name][instance_id] = service_instance
            print(f"Registered service {service_name} instance {instance_id}")
            return True
    
    def deregister_service(self, service_name: str, instance_id: str) -> bool:
        """Deregister service instance"""
        with self._lock:
            if (service_name in self.services and 
                instance_id in self.services[service_name]):
                
                del self.services[service_name][instance_id]
                
                # Remove service if no instances left
                if not self.services[service_name]:
                    del self.services[service_name]
                
                print(f"Deregistered service {service_name} instance {instance_id}")
                return True
            
            return False
    
    def heartbeat(self, service_name: str, instance_id: str) -> bool:
        """Update heartbeat for service instance"""
        with self._lock:
            if (service_name in self.services and 
                instance_id in self.services[service_name]):
                
                instance = self.services[service_name][instance_id]
                instance.last_heartbeat = time.time()
                instance.status = "UP"
                return True
            
            return False
    
    def discover_services(self, service_name: str) -> List[ServiceInstance]:
        """Discover healthy instances of a service"""
        with self._lock:
            if service_name not in self.services:
                return []
            
            healthy_instances = []
            current_time = time.time()
            
            for instance in self.services[service_name].values():
                # Check if instance is healthy based on heartbeat
                if (instance.status == "UP" and 
                    current_time - instance.last_heartbeat <= self.heartbeat_interval * 2):
                    healthy_instances.append(instance)
            
            return healthy_instances
    
    def get_all_services(self) -> Dict[str, List[ServiceInstance]]:
        """Get all registered services"""
        with self._lock:
            all_services = {}
            for service_name in self.services:
                all_services[service_name] = self.discover_services(service_name)
            return all_services
    
    def _cleanup_loop(self):
        """Background cleanup of stale instances"""
        while self.running:
            try:
                self._cleanup_stale_instances()
                time.sleep(self.cleanup_interval)
            except Exception as e:
                print(f"Error in cleanup loop: {e}")
                time.sleep(self.cleanup_interval)
    
    def _cleanup_stale_instances(self):
        """Remove instances that haven't sent heartbeat"""
        current_time = time.time()
        stale_threshold = self.heartbeat_interval * 3  # 3x heartbeat interval
        
        with self._lock:
            services_to_remove = []
            
            for service_name, instances in self.services.items():
                instances_to_remove = []
                
                for instance_id, instance in instances.items():
                    if current_time - instance.last_heartbeat > stale_threshold:
                        instances_to_remove.append(instance_id)
                
                # Remove stale instances
                for instance_id in instances_to_remove:
                    del instances[instance_id]
                    print(f"Removed stale instance {service_name}:{instance_id}")
                
                # Mark service for removal if no instances left
                if not instances:
                    services_to_remove.append(service_name)
            
            # Remove empty services
            for service_name in services_to_remove:
                del self.services[service_name]

class ServiceDiscoveryClient:
    """Client for service discovery"""
    
    def __init__(self, registry: ServiceRegistry, service_instance: ServiceInstance):
        self.registry = registry
        self.service_instance = service_instance
        self.registered = False
        
        # Heartbeat thread
        self.heartbeat_thread = threading.Thread(target=self._heartbeat_loop, daemon=True)
        self.running = False
    
    def start(self):
        """Start service discovery client"""
        # Register service
        self.registry.register_service(self.service_instance)
        self.registered = True
        
        # Start heartbeat
        self.running = True
        self.heartbeat_thread.start()
    
    def stop(self):
        """Stop service discovery client"""
        self.running = False
        
        if self.registered:
            self.registry.deregister_service(
                self.service_instance.service_name,
                self.service_instance.instance_id
            )
        
        self.heartbeat_thread.join()
    
    def discover_service(self, service_name: str) -> List[ServiceInstance]:
        """Discover instances of a service"""
        return self.registry.discover_services(service_name)
    
    def _heartbeat_loop(self):
        """Send periodic heartbeats"""
        while self.running:
            try:
                self.registry.heartbeat(
                    self.service_instance.service_name,
                    self.service_instance.instance_id
                )
                time.sleep(self.registry.heartbeat_interval)
            except Exception as e:
                print(f"Error sending heartbeat: {e}")
                time.sleep(5)
```

## 📊 Monitoring and Observability

### Distributed Tracing

```python
import uuid
import time
import threading
from typing import Dict, List, Optional
from dataclasses import dataclass, field

@dataclass
class Span:
    trace_id: str
    span_id: str
    parent_span_id: Optional[str]
    operation_name: str
    service_name: str
    start_time: float
    end_time: Optional[float] = None
    duration: Optional[float] = None
    tags: Dict = field(default_factory=dict)
    logs: List = field(default_factory=list)
    status: str = "OK"

class DistributedTracer:
    """Distributed tracing implementation"""
    
    def __init__(self):
        self.spans = {}  # span_id -> Span
        self.traces = {}  # trace_id -> List[Span]
        self._local = threading.local()
    
    def start_span(self, operation_name: str, service_name: str,
                   parent_span_id: str = None, trace_id: str = None) -> Span:
        """Start a new span"""
        if trace_id is None:
            trace_id = str(uuid.uuid4())
        
        span = Span(
            trace_id=trace_id,
            span_id=str(uuid.uuid4()),
            parent_span_id=parent_span_id,
            operation_name=operation_name,
            service_name=service_name,
            start_time=time.time()
        )
        
        self.spans[span.span_id] = span
        
        if trace_id not in self.traces:
            self.traces[trace_id] = []
        self.traces[trace_id].append(span)
        
        # Set as current span
        self._local.current_span = span
        
        return span
    
    def finish_span(self, span: Span, status: str = "OK"):
        """Finish a span"""
        span.end_time = time.time()
        span.duration = span.end_time - span.start_time
        span.status = status
        
        # Clear current span if this was it
        if hasattr(self._local, 'current_span') and self._local.current_span == span:
            self._local.current_span = None
    
    def get_current_span(self) -> Optional[Span]:
        """Get current active span"""
        return getattr(self._local, 'current_span', None)
    
    def add_tag(self, span: Span, key: str, value: str):
        """Add tag to span"""
        span.tags[key] = value
    
    def add_log(self, span: Span, message: str, level: str = "INFO"):
        """Add log entry to span"""
        span.logs.append({
            'timestamp': time.time(),
            'level': level,
            'message': message
        })
    
    def get_trace(self, trace_id: str) -> List[Span]:
        """Get all spans for a trace"""
        return self.traces.get(trace_id, [])

# Tracing decorators
def trace_function(operation_name: str, service_name: str):
    """Decorator to trace function calls"""
    def decorator(func):
        def wrapper(*args, **kwargs):
            tracer = get_tracer()
            current_span = tracer.get_current_span()
            parent_span_id = current_span.span_id if current_span else None
            trace_id = current_span.trace_id if current_span else None
            
            span = tracer.start_span(
                operation_name=operation_name,
                service_name=service_name,
                parent_span_id=parent_span_id,
                trace_id=trace_id
            )
            
            try:
                result = func(*args, **kwargs)
                tracer.finish_span(span, "OK")
                return result
            except Exception as e:
                tracer.add_tag(span, "error", "true")
                tracer.add_log(span, f"Error: {str(e)}", "ERROR")
                tracer.finish_span(span, "ERROR")
                raise
        
        return wrapper
    return decorator

# Global tracer instance
_tracer = DistributedTracer()

def get_tracer() -> DistributedTracer:
    return _tracer

# Usage example
class TracedOrderService:
    def __init__(self):
        self.service_name = "order-service"
    
    @trace_function("create_order", "order-service")
    def create_order(self, order_data: Dict) -> Dict:
        tracer = get_tracer()
        current_span = tracer.get_current_span()
        
        # Add tags
        tracer.add_tag(current_span, "user_id", str(order_data['user_id']))
        tracer.add_tag(current_span, "order_amount", str(order_data.get('total', 0)))
        
        # Validate user (creates child span)
        user = self._validate_user(order_data['user_id'])
        
        # Create order
        order = {
            'id': str(uuid.uuid4()),
            'user_id': order_data['user_id'],
            'status': 'created',
            'created_at': time.time()
        }
        
        tracer.add_log(current_span, f"Order created with ID: {order['id']}")
        return order
    
    @trace_function("validate_user", "order-service")
    def _validate_user(self, user_id: str) -> Dict:
        # Simulate user validation
        time.sleep(0.1)  # Simulate network call
        return {'id': user_id, 'valid': True}
```

## 🎯 Best Practices

### 1. Service Design Principles
- **Single Responsibility**: Each service owns one business capability
- **Autonomous**: Services can be developed, deployed, and scaled independently
- **Business-Oriented**: Organize around business capabilities, not technical layers
- **Decentralized**: Avoid centralized data management and governance

### 2. Data Management
```python
# Example of proper data boundaries
class OrderService:
    # Own order data
    def get_order(self, order_id: str) -> Dict:
        return self.order_repository.find_by_id(order_id)
    
    # Don't own user data - call user service
    def get_order_with_user(self, order_id: str) -> Dict:
        order = self.get_order(order_id)
        if order:
            user = self.user_service.get_user(order['user_id'])
            order['user'] = user
        return order
```

### 3. API Design
```python
# RESTful API design
class OrderAPI:
    def __init__(self, order_service: OrderService):
        self.order_service = order_service
    
    # GET /orders/{order_id}
    def get_order(self, order_id: str) -> Dict:
        order = self.order_service.get_order(order_id)
        if not order:
            raise NotFoundError(f"Order {order_id} not found")
        return order
    
    # POST /orders
    def create_order(self, order_data: Dict) -> Dict:
        return self.order_service.create_order(order_data)
    
    # PUT /orders/{order_id}/status
    def update_order_status(self, order_id: str, status_data: Dict) -> Dict:
        success = self.order_service.update_order_status(
            order_id, status_data['status']
        )
        if not success:
            raise NotFoundError(f"Order {order_id} not found")
        return {'status': 'updated'}
```

### 4. Deployment Strategy
```yaml
# Example Docker Compose for microservices
version: '3.8'
services:
  user-service:
    image: user-service:latest
    ports:
      - "8001:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@user-db:5432/users
    depends_on:
      - user-db
      - service-registry
  
  order-service:
    image: order-service:latest
    ports:
      - "8002:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@order-db:5432/orders
      - USER_SERVICE_URL=http://user-service:8000
    depends_on:
      - order-db
      - service-registry
  
  service-registry:
    image: consul:latest
    ports:
      - "8500:8500"
```

---

**Key Takeaway**: Microservices architecture provides benefits like independent deployment, technology diversity, and fault isolation, but introduces complexity in service communication, data consistency, and operational overhead. Success requires careful service boundary design, robust communication patterns, and comprehensive monitoring.
