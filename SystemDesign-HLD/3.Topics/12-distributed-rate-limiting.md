# Distributed Rate Limiting

## Overview

Distributed rate limiting is a critical technique for protecting systems from overload, ensuring fair resource allocation, and maintaining quality of service across distributed architectures. Unlike simple rate limiting on a single server, distributed rate limiting requires coordination across multiple nodes while maintaining performance and accuracy. This section covers essential concepts, capacity estimation, API contracts, and implementation strategies.

## Essential Concepts

### What is Rate Limiting?

Rate limiting is a technique used to control the rate at which requests are processed by a system, preventing overload and ensuring fair usage among clients.

#### Core Purposes:
- **Prevent Overload**: Protect systems from being overwhelmed by too many requests
- **Ensure Fair Usage**: Prevent any single client from monopolizing resources
- **Maintain Quality of Service**: Ensure consistent performance for all users
- **Cost Control**: Limit resource consumption and associated costs
- **Security**: Mitigate DDoS attacks and abuse

### Types of Rate Limiting

#### 1. Request Rate Limiting
```
Limit: 100 requests per minute per user
Example: API calls, web requests, database queries
```

#### 2. Bandwidth Rate Limiting
```
Limit: 10 MB/s per connection
Example: File downloads, video streaming, data transfers
```

#### 3. Concurrent Connection Limiting
```
Limit: 5 simultaneous connections per user
Example: Database connections, WebSocket connections
```

#### 4. Resource-Based Rate Limiting
```
Limit: 1000 CPU seconds per hour per user
Example: Compute-intensive operations, AI model inference
```

### Rate Limiting Algorithms

#### 1. Token Bucket Algorithm

The token bucket algorithm maintains a bucket of tokens that are consumed by requests and refilled at a constant rate.

```python
import time
import threading

class TokenBucket:
    def __init__(self, capacity, refill_rate):
        """
        capacity: Maximum number of tokens in the bucket
        refill_rate: Number of tokens added per second
        """
        self.capacity = capacity
        self.tokens = capacity
        self.refill_rate = refill_rate
        self.last_refill = time.time()
        self.lock = threading.Lock()
        
    def consume(self, tokens=1):
        """Try to consume tokens from the bucket"""
        with self.lock:
            self._refill()
            
            if self.tokens >= tokens:
                self.tokens -= tokens
                return True
            return False
            
    def _refill(self):
        """Refill tokens based on elapsed time"""
        now = time.time()
        elapsed = now - self.last_refill
        tokens_to_add = elapsed * self.refill_rate
        
        self.tokens = min(self.capacity, self.tokens + tokens_to_add)
        self.last_refill = now
        
    def available_tokens(self):
        """Get current number of available tokens"""
        with self.lock:
            self._refill()
            return self.tokens

# Usage example
bucket = TokenBucket(capacity=10, refill_rate=1)  # 10 tokens max, 1 token/second

def handle_request():
    if bucket.consume(1):
        print("Request allowed")
        # Process request
    else:
        print("Request rate limited")
        # Return 429 Too Many Requests
```

#### 2. Leaky Bucket Algorithm

The leaky bucket algorithm processes requests at a constant rate, smoothing out bursts.

```python
import time
import threading
from collections import deque

class LeakyBucket:
    def __init__(self, capacity, leak_rate):
        """
        capacity: Maximum number of requests that can be queued
        leak_rate: Number of requests processed per second
        """
        self.capacity = capacity
        self.leak_rate = leak_rate
        self.queue = deque()
        self.last_leak = time.time()
        self.lock = threading.Lock()
        
    def add_request(self, request):
        """Try to add a request to the bucket"""
        with self.lock:
            self._leak()
            
            if len(self.queue) < self.capacity:
                self.queue.append(request)
                return True
            return False
            
    def _leak(self):
        """Process requests at the leak rate"""
        now = time.time()
        elapsed = now - self.last_leak
        requests_to_process = int(elapsed * self.leak_rate)
        
        for _ in range(min(requests_to_process, len(self.queue))):
            request = self.queue.popleft()
            # Process request here
            print(f"Processing request: {request}")
            
        self.last_leak = now
        
    def pending_requests(self):
        """Get number of pending requests"""
        with self.lock:
            return len(self.queue)

# Usage example
bucket = LeakyBucket(capacity=5, leak_rate=2)  # 5 requests max, 2 requests/second

def handle_request(request_id):
    if bucket.add_request(request_id):
        print(f"Request {request_id} queued")
    else:
        print(f"Request {request_id} rejected - bucket full")
```

#### 3. Fixed Window Algorithm

The fixed window algorithm counts requests in fixed time windows.

```python
import time
import threading
from collections import defaultdict

class FixedWindowRateLimiter:
    def __init__(self, limit, window_size):
        """
        limit: Maximum requests per window
        window_size: Window size in seconds
        """
        self.limit = limit
        self.window_size = window_size
        self.windows = defaultdict(int)  # window_start -> request_count
        self.lock = threading.Lock()
        
    def is_allowed(self, client_id):
        """Check if request is allowed for client"""
        with self.lock:
            current_window = self._get_current_window()
            key = f"{client_id}:{current_window}"
            
            if self.windows[key] < self.limit:
                self.windows[key] += 1
                return True
            return False
            
    def _get_current_window(self):
        """Get current window start time"""
        now = time.time()
        return int(now // self.window_size) * self.window_size
        
    def cleanup_old_windows(self):
        """Remove old window data"""
        current_window = self._get_current_window()
        cutoff = current_window - self.window_size
        
        keys_to_remove = [
            key for key in self.windows.keys()
            if int(key.split(':')[1]) < cutoff
        ]
        
        for key in keys_to_remove:
            del self.windows[key]

# Usage example
limiter = FixedWindowRateLimiter(limit=100, window_size=60)  # 100 requests per minute

def handle_request(client_id):
    if limiter.is_allowed(client_id):
        print(f"Request from {client_id} allowed")
        # Process request
    else:
        print(f"Request from {client_id} rate limited")
        # Return 429 Too Many Requests
```

#### 4. Sliding Window Log Algorithm

The sliding window log algorithm maintains a log of request timestamps for precise rate limiting.

```python
import time
import threading
from collections import defaultdict, deque

class SlidingWindowLogRateLimiter:
    def __init__(self, limit, window_size):
        """
        limit: Maximum requests per window
        window_size: Window size in seconds
        """
        self.limit = limit
        self.window_size = window_size
        self.request_logs = defaultdict(deque)  # client_id -> deque of timestamps
        self.lock = threading.Lock()
        
    def is_allowed(self, client_id):
        """Check if request is allowed for client"""
        with self.lock:
            now = time.time()
            cutoff = now - self.window_size
            
            # Remove old requests
            while (self.request_logs[client_id] and 
                   self.request_logs[client_id][0] <= cutoff):
                self.request_logs[client_id].popleft()
                
            # Check if under limit
            if len(self.request_logs[client_id]) < self.limit:
                self.request_logs[client_id].append(now)
                return True
            return False
            
    def get_request_count(self, client_id):
        """Get current request count for client"""
        with self.lock:
            now = time.time()
            cutoff = now - self.window_size
            
            # Remove old requests
            while (self.request_logs[client_id] and 
                   self.request_logs[client_id][0] <= cutoff):
                self.request_logs[client_id].popleft()
                
            return len(self.request_logs[client_id])

# Usage example
limiter = SlidingWindowLogRateLimiter(limit=100, window_size=60)  # 100 requests per minute

def handle_request(client_id):
    if limiter.is_allowed(client_id):
        print(f"Request from {client_id} allowed")
        # Process request
    else:
        print(f"Request from {client_id} rate limited")
        # Return 429 Too Many Requests
```

#### 5. Sliding Window Counter Algorithm

The sliding window counter algorithm approximates sliding window behavior with less memory usage.

```python
import time
import threading
from collections import defaultdict

class SlidingWindowCounterRateLimiter:
    def __init__(self, limit, window_size, num_buckets=10):
        """
        limit: Maximum requests per window
        window_size: Window size in seconds
        num_buckets: Number of sub-windows for approximation
        """
        self.limit = limit
        self.window_size = window_size
        self.bucket_size = window_size / num_buckets
        self.num_buckets = num_buckets
        self.buckets = defaultdict(lambda: defaultdict(int))  # client_id -> bucket -> count
        self.lock = threading.Lock()
        
    def is_allowed(self, client_id):
        """Check if request is allowed for client"""
        with self.lock:
            now = time.time()
            current_bucket = int(now // self.bucket_size)
            
            # Calculate weighted count
            total_count = 0
            for i in range(self.num_buckets):
                bucket_id = current_bucket - i
                if bucket_id in self.buckets[client_id]:
                    # Weight based on how much of the bucket is within the window
                    if i == 0:
                        # Current bucket - partial weight
                        elapsed_in_bucket = now % self.bucket_size
                        weight = elapsed_in_bucket / self.bucket_size
                    else:
                        # Full weight for complete buckets within window
                        weight = 1.0
                        
                    total_count += self.buckets[client_id][bucket_id] * weight
                    
            if total_count < self.limit:
                self.buckets[client_id][current_bucket] += 1
                return True
            return False
            
    def cleanup_old_buckets(self):
        """Remove old bucket data"""
        now = time.time()
        current_bucket = int(now // self.bucket_size)
        cutoff_bucket = current_bucket - self.num_buckets
        
        for client_id in self.buckets:
            buckets_to_remove = [
                bucket_id for bucket_id in self.buckets[client_id]
                if bucket_id < cutoff_bucket
            ]
            for bucket_id in buckets_to_remove:
                del self.buckets[client_id][bucket_id]

# Usage example
limiter = SlidingWindowCounterRateLimiter(limit=100, window_size=60, num_buckets=6)

def handle_request(client_id):
    if limiter.is_allowed(client_id):
        print(f"Request from {client_id} allowed")
        # Process request
    else:
        print(f"Request from {client_id} rate limited")
        # Return 429 Too Many Requests
```

## Capacity Estimation

Capacity estimation is crucial for setting appropriate rate limits that balance system protection with user experience.

### Factors to Consider

#### 1. System Resources
```python
class SystemCapacityEstimator:
    def __init__(self):
        self.cpu_cores = 8
        self.memory_gb = 32
        self.network_bandwidth_mbps = 1000
        self.disk_iops = 10000
        
    def estimate_request_capacity(self, request_profile):
        """Estimate maximum requests per second based on resource constraints"""
        
        # CPU constraint
        cpu_rps = (self.cpu_cores * 1000) / request_profile['cpu_ms_per_request']
        
        # Memory constraint
        memory_rps = (self.memory_gb * 1024 * 1024 * 1024) / request_profile['memory_bytes_per_request']
        
        # Network constraint
        network_rps = (self.network_bandwidth_mbps * 1024 * 1024 / 8) / request_profile['network_bytes_per_request']
        
        # Disk I/O constraint
        disk_rps = self.disk_iops / request_profile['disk_operations_per_request']
        
        # The bottleneck determines capacity
        bottleneck_rps = min(cpu_rps, memory_rps, network_rps, disk_rps)
        
        # Add safety margin (typically 70-80% of theoretical maximum)
        safe_rps = bottleneck_rps * 0.75
        
        return {
            'theoretical_max_rps': bottleneck_rps,
            'safe_max_rps': safe_rps,
            'bottleneck': self._identify_bottleneck(cpu_rps, memory_rps, network_rps, disk_rps),
            'breakdown': {
                'cpu_limited_rps': cpu_rps,
                'memory_limited_rps': memory_rps,
                'network_limited_rps': network_rps,
                'disk_limited_rps': disk_rps
            }
        }
        
    def _identify_bottleneck(self, cpu_rps, memory_rps, network_rps, disk_rps):
        """Identify the primary bottleneck"""
        constraints = {
            'cpu': cpu_rps,
            'memory': memory_rps,
            'network': network_rps,
            'disk': disk_rps
        }
        return min(constraints, key=constraints.get)

# Usage example
estimator = SystemCapacityEstimator()

# Profile for a typical API request
api_request_profile = {
    'cpu_ms_per_request': 10,      # 10ms CPU time
    'memory_bytes_per_request': 1024 * 1024,  # 1MB memory
    'network_bytes_per_request': 1024,  # 1KB network
    'disk_operations_per_request': 2   # 2 disk operations
}

capacity = estimator.estimate_request_capacity(api_request_profile)
print(f"Safe maximum RPS: {capacity['safe_max_rps']:.0f}")
print(f"Primary bottleneck: {capacity['bottleneck']}")
```

#### 2. User Behavior Analysis
```python
import numpy as np
from collections import defaultdict

class UserBehaviorAnalyzer:
    def __init__(self):
        self.request_logs = []  # List of (timestamp, user_id, endpoint) tuples
        
    def analyze_traffic_patterns(self, logs):
        """Analyze traffic patterns from historical logs"""
        self.request_logs = logs
        
        return {
            'peak_rps': self._calculate_peak_rps(),
            'average_rps': self._calculate_average_rps(),
            'user_distribution': self._analyze_user_distribution(),
            'endpoint_distribution': self._analyze_endpoint_distribution(),
            'time_patterns': self._analyze_time_patterns()
        }
        
    def _calculate_peak_rps(self):
        """Calculate peak requests per second"""
        if not self.request_logs:
            return 0
            
        # Group requests by second
        requests_per_second = defaultdict(int)
        for timestamp, _, _ in self.request_logs:
            second = int(timestamp)
            requests_per_second[second] += 1
            
        return max(requests_per_second.values()) if requests_per_second else 0
        
    def _calculate_average_rps(self):
        """Calculate average requests per second"""
        if not self.request_logs:
            return 0
            
        if len(self.request_logs) < 2:
            return 0
            
        start_time = min(log[0] for log in self.request_logs)
        end_time = max(log[0] for log in self.request_logs)
        duration = end_time - start_time
        
        return len(self.request_logs) / duration if duration > 0 else 0
        
    def _analyze_user_distribution(self):
        """Analyze request distribution per user"""
        user_requests = defaultdict(int)
        for _, user_id, _ in self.request_logs:
            user_requests[user_id] += 1
            
        request_counts = list(user_requests.values())
        
        return {
            'total_users': len(user_requests),
            'avg_requests_per_user': np.mean(request_counts) if request_counts else 0,
            'median_requests_per_user': np.median(request_counts) if request_counts else 0,
            'p95_requests_per_user': np.percentile(request_counts, 95) if request_counts else 0,
            'p99_requests_per_user': np.percentile(request_counts, 99) if request_counts else 0
        }
        
    def _analyze_endpoint_distribution(self):
        """Analyze request distribution per endpoint"""
        endpoint_requests = defaultdict(int)
        for _, _, endpoint in self.request_logs:
            endpoint_requests[endpoint] += 1
            
        total_requests = len(self.request_logs)
        
        return {
            endpoint: {
                'count': count,
                'percentage': (count / total_requests) * 100
            }
            for endpoint, count in endpoint_requests.items()
        }
        
    def _analyze_time_patterns(self):
        """Analyze traffic patterns over time"""
        hourly_requests = defaultdict(int)
        daily_requests = defaultdict(int)
        
        for timestamp, _, _ in self.request_logs:
            hour = int(timestamp) // 3600 % 24  # Hour of day (0-23)
            day = int(timestamp) // 86400 % 7   # Day of week (0-6)
            
            hourly_requests[hour] += 1
            daily_requests[day] += 1
            
        return {
            'hourly_distribution': dict(hourly_requests),
            'daily_distribution': dict(daily_requests),
            'peak_hour': max(hourly_requests, key=hourly_requests.get) if hourly_requests else 0,
            'peak_day': max(daily_requests, key=daily_requests.get) if daily_requests else 0
        }

# Usage example
analyzer = UserBehaviorAnalyzer()

# Sample log data (timestamp, user_id, endpoint)
sample_logs = [
    (1640995200, 'user1', '/api/users'),
    (1640995201, 'user2', '/api/orders'),
    (1640995202, 'user1', '/api/products'),
    # ... more log entries
]

patterns = analyzer.analyze_traffic_patterns(sample_logs)
print(f"Peak RPS: {patterns['peak_rps']}")
print(f"Average RPS: {patterns['average_rps']:.2f}")
print(f"P95 requests per user: {patterns['user_distribution']['p95_requests_per_user']:.0f}")
```

#### 3. Business Requirements
```python
class BusinessRequirementsAnalyzer:
    def __init__(self):
        self.service_tiers = {
            'free': {'requests_per_hour': 1000, 'burst_allowance': 10},
            'basic': {'requests_per_hour': 10000, 'burst_allowance': 50},
            'premium': {'requests_per_hour': 100000, 'burst_allowance': 200},
            'enterprise': {'requests_per_hour': 1000000, 'burst_allowance': 1000}
        }
        
    def calculate_rate_limits(self, user_tier, endpoint_type):
        """Calculate rate limits based on business requirements"""
        base_limits = self.service_tiers.get(user_tier, self.service_tiers['free'])
        
        # Adjust limits based on endpoint type
        endpoint_multipliers = {
            'read': 1.0,      # Normal rate for read operations
            'write': 0.5,     # Stricter limits for write operations
            'compute': 0.1,   # Very strict limits for compute-intensive operations
            'admin': 0.2      # Strict limits for admin operations
        }
        
        multiplier = endpoint_multipliers.get(endpoint_type, 1.0)
        
        return {
            'requests_per_hour': int(base_limits['requests_per_hour'] * multiplier),
            'requests_per_minute': int(base_limits['requests_per_hour'] * multiplier / 60),
            'requests_per_second': int(base_limits['requests_per_hour'] * multiplier / 3600),
            'burst_allowance': int(base_limits['burst_allowance'] * multiplier)
        }
        
    def estimate_infrastructure_needs(self, expected_users_by_tier):
        """Estimate infrastructure needs based on user distribution"""
        total_rps = 0
        total_users = 0
        
        for tier, user_count in expected_users_by_tier.items():
            if tier in self.service_tiers:
                tier_rps = (self.service_tiers[tier]['requests_per_hour'] / 3600) * user_count
                total_rps += tier_rps
                total_users += user_count
                
        # Add peak traffic multiplier (typically 3-5x average)
        peak_rps = total_rps * 4
        
        return {
            'average_rps': total_rps,
            'peak_rps': peak_rps,
            'total_users': total_users,
            'recommended_server_capacity': peak_rps * 1.2,  # 20% safety margin
            'estimated_monthly_requests': total_rps * 86400 * 30
        }

# Usage example
analyzer = BusinessRequirementsAnalyzer()

# Calculate rate limits for different user tiers and endpoint types
free_user_read_limits = analyzer.calculate_rate_limits('free', 'read')
premium_user_write_limits = analyzer.calculate_rate_limits('premium', 'write')

print(f"Free user read limits: {free_user_read_limits}")
print(f"Premium user write limits: {premium_user_write_limits}")

# Estimate infrastructure needs
expected_users = {
    'free': 10000,
    'basic': 1000,
    'premium': 100,
    'enterprise': 10
}

infrastructure_needs = analyzer.estimate_infrastructure_needs(expected_users)
print(f"Estimated peak RPS: {infrastructure_needs['peak_rps']:.0f}")
print(f"Recommended server capacity: {infrastructure_needs['recommended_server_capacity']:.0f} RPS")
```

## API Contracts

API contracts define the rate limiting behavior, error responses, and client expectations for distributed rate limiting systems.

### Rate Limiting Headers

Standard HTTP headers provide clients with rate limiting information:

```python
class RateLimitHeaders:
    """Standard rate limiting headers for HTTP APIs"""
    
    @staticmethod
    def generate_headers(limit, remaining, reset_time, retry_after=None):
        """Generate standard rate limiting headers"""
        headers = {
            'X-RateLimit-Limit': str(limit),           # Total limit
            'X-RateLimit-Remaining': str(remaining),    # Remaining requests
            'X-RateLimit-Reset': str(int(reset_time)), # Reset timestamp
        }
        
        if retry_after:
            headers['Retry-After'] = str(retry_after)  # Seconds to wait
            
        return headers
    
    @staticmethod
    def generate_github_style_headers(limit, remaining, reset_time):
        """Generate GitHub-style rate limiting headers"""
        return {
            'X-RateLimit-Limit': str(limit),
            'X-RateLimit-Remaining': str(remaining),
            'X-RateLimit-Reset': str(int(reset_time)),
            'X-RateLimit-Used': str(limit - remaining)
        }
    
    @staticmethod
    def generate_twitter_style_headers(limit, remaining, reset_time):
        """Generate Twitter-style rate limiting headers"""
        return {
            'x-rate-limit-limit': str(limit),
            'x-rate-limit-remaining': str(remaining),
            'x-rate-limit-reset': str(int(reset_time))
        }

# Usage in API response
def api_endpoint_with_rate_limiting(request):
    client_id = get_client_id(request)
    
    # Check rate limit
    rate_limiter = get_rate_limiter()
    limit_info = rate_limiter.check_limit(client_id)
    
    if limit_info['allowed']:
        # Process request
        response_data = process_request(request)
        
        # Add rate limiting headers
        headers = RateLimitHeaders.generate_headers(
            limit=limit_info['limit'],
            remaining=limit_info['remaining'],
            reset_time=limit_info['reset_time']
        )
        
        return create_response(200, response_data, headers)
    else:
        # Rate limited
        headers = RateLimitHeaders.generate_headers(
            limit=limit_info['limit'],
            remaining=0,
            reset_time=limit_info['reset_time'],
            retry_after=limit_info['retry_after']
        )
        
        error_response = {
            'error': 'Rate limit exceeded',
            'message': f'Too many requests. Limit: {limit_info["limit"]} per {limit_info["window"]}',
            'retry_after': limit_info['retry_after']
        }
        
        return create_response(429, error_response, headers)
```

### Error Response Formats

Standardized error responses help clients handle rate limiting gracefully:

```python
class RateLimitErrorResponses:
    """Standard error response formats for rate limiting"""
    
    @staticmethod
    def rfc6585_format(limit, window, retry_after):
        """RFC 6585 compliant error response"""
        return {
            'error': {
                'code': 429,
                'message': 'Too Many Requests',
                'description': f'Rate limit of {limit} requests per {window} exceeded',
                'retry_after': retry_after
            }
        }
    
    @staticmethod
    def json_api_format(limit, window, retry_after):
        """JSON API specification format"""
        return {
            'errors': [{
                'status': '429',
                'code': 'RATE_LIMIT_EXCEEDED',
                'title': 'Rate limit exceeded',
                'detail': f'You have exceeded the rate limit of {limit} requests per {window}',
                'meta': {
                    'retry_after': retry_after,
                    'limit': limit,
                    'window': window
                }
            }]
        }
    
    @staticmethod
    def problem_details_format(limit, window, retry_after):
        """RFC 7807 Problem Details format"""
        return {
            'type': 'https://example.com/problems/rate-limit-exceeded',
            'title': 'Rate limit exceeded',
            'status': 429,
            'detail': f'The rate limit of {limit} requests per {window} has been exceeded',
            'instance': '/api/v1/users/123',
            'retry_after': retry_after,
            'rate_limit': {
                'limit': limit,
                'window': window
            }
        }
    
    @staticmethod
    def custom_format(limit, window, retry_after, additional_info=None):
        """Custom error format with additional information"""
        response = {
            'success': False,
            'error_code': 'RATE_LIMIT_EXCEEDED',
            'message': 'Rate limit exceeded',
            'details': {
                'limit': limit,
                'window': window,
                'retry_after': retry_after,
                'timestamp': time.time()
            }
        }
        
        if additional_info:
            response['details'].update(additional_info)
            
        return response

# Usage example
def handle_rate_limit_exceeded(limit, window, retry_after, format_type='rfc6585'):
    """Generate appropriate error response based on format type"""
    
    formatters = {
        'rfc6585': RateLimitErrorResponses.rfc6585_format,
        'json_api': RateLimitErrorResponses.json_api_format,
        'problem_details': RateLimitErrorResponses.problem_details_format,
        'custom': RateLimitErrorResponses.custom_format
    }
    
    formatter = formatters.get(format_type, RateLimitErrorResponses.rfc6585_format)
    return formatter(limit, window, retry_after)
```

### Client-Side Rate Limiting

Implementing client-side rate limiting helps reduce server load and improves user experience:

```python
import time
import asyncio
from collections import deque

class ClientSideRateLimiter:
    """Client-side rate limiter to respect server limits"""
    
    def __init__(self, requests_per_second, burst_size=None):
        self.requests_per_second = requests_per_second
        self.burst_size = burst_size or requests_per_second
        self.tokens = self.burst_size
        self.last_refill = time.time()
        self.request_queue = deque()
        
    async def acquire(self):
        """Acquire permission to make a request"""
        while True:
            if self._try_acquire():
                return
            
            # Wait before trying again
            await asyncio.sleep(0.1)
            
    def _try_acquire(self):
        """Try to acquire a token"""
        now = time.time()
        elapsed = now - self.last_refill
        
        # Refill tokens
        tokens_to_add = elapsed * self.requests_per_second
        self.tokens = min(self.burst_size, self.tokens + tokens_to_add)
        self.last_refill = now
        
        # Try to consume a token
        if self.tokens >= 1:
            self.tokens -= 1
            return True
        return False
        
    async def make_request(self, request_func, *args, **kwargs):
        """Make a rate-limited request"""
        await self.acquire()
        return await request_func(*args, **kwargs)

class AdaptiveClientRateLimiter:
    """Adaptive client-side rate limiter that adjusts based on server responses"""
    
    def __init__(self, initial_rate=10):
        self.current_rate = initial_rate
        self.min_rate = 1
        self.max_rate = 1000
        self.success_count = 0
        self.failure_count = 0
        self.last_adjustment = time.time()
        self.adjustment_interval = 60  # Adjust every minute
        
    def handle_response(self, status_code, headers):
        """Handle server response and adjust rate if needed"""
        if status_code == 429:  # Rate limited
            self.failure_count += 1
            self._decrease_rate()
        elif 200 <= status_code < 300:  # Success
            self.success_count += 1
            
        # Periodic adjustment based on success/failure ratio
        if time.time() - self.last_adjustment > self.adjustment_interval:
            self._adjust_rate_based_on_performance()
            
    def _decrease_rate(self):
        """Decrease rate immediately when rate limited"""
        self.current_rate = max(self.min_rate, self.current_rate * 0.5)
        print(f"Rate limited - decreased rate to {self.current_rate}")
        
    def _adjust_rate_based_on_performance(self):
        """Adjust rate based on success/failure ratio"""
        total_requests = self.success_count + self.failure_count
        
        if total_requests > 0:
            success_rate = self.success_count / total_requests
            
            if success_rate > 0.95:  # High success rate - can increase
                self.current_rate = min(self.max_rate, self.current_rate * 1.1)
            elif success_rate < 0.8:  # Low success rate - decrease
                self.current_rate = max(self.min_rate, self.current_rate * 0.9)
                
        # Reset counters
        self.success_count = 0
        self.failure_count = 0
        self.last_adjustment = time.time()
        
    def get_current_rate(self):
        """Get current rate limit"""
        return self.current_rate

# Usage example
async def example_client_usage():
    # Simple rate limiter
    rate_limiter = ClientSideRateLimiter(requests_per_second=10, burst_size=20)
    
    async def make_api_call():
        # Simulate API call
        await asyncio.sleep(0.1)
        return "API response"
    
    # Make rate-limited requests
    for i in range(50):
        response = await rate_limiter.make_request(make_api_call)
        print(f"Request {i}: {response}")
        
    # Adaptive rate limiter
    adaptive_limiter = AdaptiveClientRateLimiter(initial_rate=10)
    
    # Simulate handling responses
    adaptive_limiter.handle_response(200, {})  # Success
    adaptive_limiter.handle_response(429, {})  # Rate limited
    adaptive_limiter.handle_response(200, {})  # Success
    
    print(f"Current adaptive rate: {adaptive_limiter.get_current_rate()}")

# Run example
# asyncio.run(example_client_usage())
```

### Service Level Agreements (SLAs)

Define clear SLAs for rate limiting behavior:

```python
class RateLimitingSLA:
    """Define and validate rate limiting SLAs"""
    
    def __init__(self):
        self.sla_definitions = {
            'free_tier': {
                'requests_per_hour': 1000,
                'burst_allowance': 10,
                'guaranteed_availability': 95.0,  # 95% availability
                'max_response_time_ms': 500,
                'rate_limit_accuracy': 98.0  # 98% accuracy in rate limiting
            },
            'paid_tier': {
                'requests_per_hour': 10000,
                'burst_allowance': 100,
                'guaranteed_availability': 99.5,  # 99.5% availability
                'max_response_time_ms': 200,
                'rate_limit_accuracy': 99.5  # 99.5% accuracy
            },
            'enterprise_tier': {
                'requests_per_hour': 100000,
                'burst_allowance': 1000,
                'guaranteed_availability': 99.9,  # 99.9% availability
                'max_response_time_ms': 100,
                'rate_limit_accuracy': 99.9  # 99.9% accuracy
            }
        }
        
    def get_sla(self, tier):
        """Get SLA for a specific tier"""
        return self.sla_definitions.get(tier, self.sla_definitions['free_tier'])
        
    def validate_sla_compliance(self, tier, metrics):
        """Validate if current metrics meet SLA requirements"""
        sla = self.get_sla(tier)
        compliance = {}
        
        # Check availability
        if 'availability_percentage' in metrics:
            compliance['availability'] = {
                'required': sla['guaranteed_availability'],
                'actual': metrics['availability_percentage'],
                'compliant': metrics['availability_percentage'] >= sla['guaranteed_availability']
            }
            
        # Check response time
        if 'avg_response_time_ms' in metrics:
            compliance['response_time'] = {
                'required': sla['max_response_time_ms'],
                'actual': metrics['avg_response_time_ms'],
                'compliant': metrics['avg_response_time_ms'] <= sla['max_response_time_ms']
            }
            
        # Check rate limiting accuracy
        if 'rate_limit_accuracy' in metrics:
            compliance['rate_limit_accuracy'] = {
                'required': sla['rate_limit_accuracy'],
                'actual': metrics['rate_limit_accuracy'],
                'compliant': metrics['rate_limit_accuracy'] >= sla['rate_limit_accuracy']
            }
            
        return compliance
        
    def calculate_sla_credits(self, tier, compliance_results):
        """Calculate SLA credits based on compliance"""
        sla = self.get_sla(tier)
        credits = 0
        
        # Availability credits
        if 'availability' in compliance_results and not compliance_results['availability']['compliant']:
            availability_shortfall = sla['guaranteed_availability'] - compliance_results['availability']['actual']
            credits += availability_shortfall * 0.1  # 0.1% credit per 1% shortfall
            
        # Response time credits
        if 'response_time' in compliance_results and not compliance_results['response_time']['compliant']:
            credits += 5  # 5% credit for response time violations
            
        return min(credits, 100)  # Cap at 100% credit

# Usage example
sla_manager = RateLimitingSLA()

# Get SLA for enterprise tier
enterprise_sla = sla_manager.get_sla('enterprise_tier')
print(f"Enterprise SLA: {enterprise_sla}")

# Validate compliance
current_metrics = {
    'availability_percentage': 99.8,
    'avg_response_time_ms': 150,
    'rate_limit_accuracy': 99.95
}

compliance = sla_manager.validate_sla_compliance('enterprise_tier', current_metrics)
print(f"SLA Compliance: {compliance}")

# Calculate credits if needed
credits = sla_manager.calculate_sla_credits('enterprise_tier', compliance)
print(f"SLA Credits: {credits}%")
```

## Conclusion

Distributed rate limiting is essential for protecting systems and ensuring fair resource allocation. Key takeaways:

1. **Choose Appropriate Algorithms**: Select rate limiting algorithms based on your specific requirements
2. **Estimate Capacity Accurately**: Use system resources, user behavior, and business requirements to set limits
3. **Design Clear API Contracts**: Provide clear headers, error responses, and SLAs
4. **Implement Client-Side Limiting**: Help reduce server load and improve user experience
5. **Monitor and Adjust**: Continuously monitor performance and adjust limits as needed

The key is to balance system protection with user experience while maintaining accuracy and performance in distributed environments.

## Next Steps

In the following sections, we'll explore:
- **Security in Distributed Systems**: Protecting rate limiting infrastructure
- **Observability**: Monitoring and alerting for rate limiting systems
- **HLD Interview Problems**: Applying rate limiting concepts in system design interviews