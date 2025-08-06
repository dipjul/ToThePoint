# Observability in Distributed Systems

## Overview

Observability is the ability to understand the internal state of a system by examining its external outputs. In distributed systems, observability becomes critical due to the complexity of interactions between multiple services, networks, and data stores. This section covers the three pillars of observability: logging, monitoring, and tracing, along with anomaly detection and root cause analysis techniques.

## Essential Concepts

### The Three Pillars of Observability

#### 1. Metrics
Numerical measurements of system behavior over time.
- **Examples**: CPU usage, memory consumption, request rate, error rate
- **Characteristics**: Aggregatable, efficient storage, good for alerting
- **Use Cases**: Performance monitoring, capacity planning, SLA tracking

#### 2. Logs
Discrete events that happened at a specific time.
- **Examples**: Application logs, access logs, error logs
- **Characteristics**: High cardinality, detailed context, searchable
- **Use Cases**: Debugging, audit trails, security analysis

#### 3. Traces
Records of requests as they flow through distributed systems.
- **Examples**: HTTP request traces, database query traces
- **Characteristics**: Shows request flow, latency breakdown, dependencies
- **Use Cases**: Performance optimization, dependency mapping, bottleneck identification

### Key Observability Concepts

#### Telemetry
The automatic collection and transmission of data from remote sources.

#### Cardinality
The number of unique values in a dataset (important for metrics storage costs).

#### Signal-to-Noise Ratio
The proportion of useful information versus irrelevant data.

#### Service Level Objectives (SLOs)
Specific, measurable targets for service performance.

## Logging and Monitoring

### Structured Logging

```python
import json
import time
import logging
from datetime import datetime
from typing import Dict, Any, Optional

class StructuredLogger:
    def __init__(self, service_name: str, version: str):
        self.service_name = service_name
        self.version = version
        self.logger = logging.getLogger(service_name)
        
        # Configure JSON formatter
        handler = logging.StreamHandler()
        handler.setFormatter(self._create_json_formatter())
        self.logger.addHandler(handler)
        self.logger.setLevel(logging.INFO)
        
    def _create_json_formatter(self):
        class JSONFormatter(logging.Formatter):
            def format(self, record):
                log_entry = {
                    'timestamp': datetime.utcnow().isoformat() + 'Z',
                    'level': record.levelname,
                    'service': record.name,
                    'message': record.getMessage(),
                    'logger': record.name,
                    'thread': record.thread,
                    'filename': record.filename,
                    'line_number': record.lineno
                }
                
                # Add extra fields if present
                if hasattr(record, 'extra_fields'):
                    log_entry.update(record.extra_fields)
                    
                return json.dumps(log_entry)
                
        return JSONFormatter()
        
    def info(self, message: str, **kwargs):
        extra = {'extra_fields': kwargs} if kwargs else {}
        self.logger.info(message, extra=extra)
        
    def error(self, message: str, error: Exception = None, **kwargs):
        extra_fields = kwargs.copy()
        if error:
            extra_fields.update({
                'error_type': type(error).__name__,
                'error_message': str(error),
                'stack_trace': self._get_stack_trace(error)
            })
        extra = {'extra_fields': extra_fields} if extra_fields else {}
        self.logger.error(message, extra=extra)
        
    def _get_stack_trace(self, error: Exception) -> str:
        import traceback
        return ''.join(traceback.format_exception(type(error), error, error.__traceback__))

# Usage example
logger = StructuredLogger('user-service', '1.0.0')

logger.info('User login attempt', 
           user_id='user123', 
           ip_address='192.168.1.100',
           user_agent='Mozilla/5.0...')

try:
    # Some operation that might fail
    result = risky_operation()
except Exception as e:
    logger.error('Operation failed', 
                error=e,
                operation='risky_operation',
                user_id='user123')
```

### Metrics Collection

```python
import time
import threading
from collections import defaultdict, deque
from typing import Dict, List, Optional
from dataclasses import dataclass
from enum import Enum

class MetricType(Enum):
    COUNTER = "counter"
    GAUGE = "gauge"
    HISTOGRAM = "histogram"
    SUMMARY = "summary"

@dataclass
class MetricSample:
    name: str
    value: float
    timestamp: float
    labels: Dict[str, str]
    metric_type: MetricType

class MetricsCollector:
    def __init__(self):
        self.metrics = defaultdict(list)
        self.counters = defaultdict(float)
        self.gauges = defaultdict(float)
        self.histograms = defaultdict(list)
        self.lock = threading.Lock()
        
    def increment_counter(self, name: str, value: float = 1.0, labels: Dict[str, str] = None):
        """Increment a counter metric"""
        with self.lock:
            key = self._create_metric_key(name, labels or {})
            self.counters[key] += value
            
            sample = MetricSample(
                name=name,
                value=self.counters[key],
                timestamp=time.time(),
                labels=labels or {},
                metric_type=MetricType.COUNTER
            )
            self.metrics[name].append(sample)
            
    def set_gauge(self, name: str, value: float, labels: Dict[str, str] = None):
        """Set a gauge metric value"""
        with self.lock:
            key = self._create_metric_key(name, labels or {})
            self.gauges[key] = value
            
            sample = MetricSample(
                name=name,
                value=value,
                timestamp=time.time(),
                labels=labels or {},
                metric_type=MetricType.GAUGE
            )
            self.metrics[name].append(sample)
            
    def observe_histogram(self, name: str, value: float, labels: Dict[str, str] = None):
        """Add an observation to a histogram"""
        with self.lock:
            key = self._create_metric_key(name, labels or {})
            self.histograms[key].append(value)
            
            sample = MetricSample(
                name=name,
                value=value,
                timestamp=time.time(),
                labels=labels or {},
                metric_type=MetricType.HISTOGRAM
            )
            self.metrics[name].append(sample)
            
    def get_metrics(self) -> Dict[str, List[MetricSample]]:
        """Get all collected metrics"""
        with self.lock:
            return dict(self.metrics)
            
    def _create_metric_key(self, name: str, labels: Dict[str, str]) -> str:
        """Create a unique key for metric with labels"""
        label_str = ','.join([f"{k}={v}" for k, v in sorted(labels.items())])
        return f"{name}{{{label_str}}}" if label_str else name

# Application Performance Monitoring
class APMTracker:
    def __init__(self, metrics_collector: MetricsCollector):
        self.metrics = metrics_collector
        self.active_requests = {}
        
    def start_request(self, request_id: str, endpoint: str, method: str):
        """Start tracking a request"""
        self.active_requests[request_id] = {
            'start_time': time.time(),
            'endpoint': endpoint,
            'method': method
        }
        
        # Increment request counter
        self.metrics.increment_counter(
            'http_requests_total',
            labels={'endpoint': endpoint, 'method': method}
        )
        
    def end_request(self, request_id: str, status_code: int):
        """End tracking a request"""
        if request_id not in self.active_requests:
            return
            
        request_info = self.active_requests.pop(request_id)
        duration = time.time() - request_info['start_time']
        
        labels = {
            'endpoint': request_info['endpoint'],
            'method': request_info['method'],
            'status_code': str(status_code)
        }
        
        # Record request duration
        self.metrics.observe_histogram('http_request_duration_seconds', duration, labels)
        
        # Update active requests gauge
        self.metrics.set_gauge('http_requests_active', len(self.active_requests))

# Usage example
metrics = MetricsCollector()
apm = APMTracker(metrics)

# Track a request
apm.start_request('req123', '/api/users', 'GET')
time.sleep(0.1)  # Simulate processing
apm.end_request('req123', 200)

# Custom metrics
metrics.increment_counter('database_queries_total', labels={'table': 'users', 'operation': 'select'})
metrics.set_gauge('memory_usage_bytes', 1024*1024*512)  # 512MB
```

## Anomaly Detection

Anomaly detection helps identify unusual patterns that might indicate problems.

### Statistical Anomaly Detection

```python
import numpy as np
from scipy import stats
from collections import deque
from typing import List, Tuple, Optional
import time

class StatisticalAnomalyDetector:
    def __init__(self, window_size: int = 100, threshold: float = 2.0):
        self.window_size = window_size
        self.threshold = threshold  # Number of standard deviations
        self.data_window = deque(maxlen=window_size)
        
    def add_data_point(self, value: float) -> bool:
        """Add data point and return True if it's an anomaly"""
        self.data_window.append(value)
        
        if len(self.data_window) < 10:  # Need minimum data points
            return False
            
        return self._is_anomaly(value)
        
    def _is_anomaly(self, value: float) -> bool:
        """Check if value is an anomaly using z-score"""
        if len(self.data_window) < 2:
            return False
            
        data_array = np.array(list(self.data_window)[:-1])  # Exclude current value
        mean = np.mean(data_array)
        std = np.std(data_array)
        
        if std == 0:  # No variation in data
            return False
            
        z_score = abs((value - mean) / std)
        return z_score > self.threshold
        
    def get_statistics(self) -> Dict[str, float]:
        """Get current window statistics"""
        if len(self.data_window) < 2:
            return {}
            
        data_array = np.array(list(self.data_window))
        return {
            'mean': np.mean(data_array),
            'std': np.std(data_array),
            'min': np.min(data_array),
            'max': np.max(data_array),
            'median': np.median(data_array)
        }

class TimeSeriesAnomalyDetector:
    def __init__(self, seasonal_period: int = 24):
        self.seasonal_period = seasonal_period
        self.historical_data = deque(maxlen=seasonal_period * 7)  # 7 periods of history
        
    def add_data_point(self, timestamp: float, value: float) -> Dict[str, any]:
        """Add data point and detect anomalies"""
        self.historical_data.append((timestamp, value))
        
        if len(self.historical_data) < self.seasonal_period * 2:
            return {'is_anomaly': False, 'reason': 'insufficient_data'}
            
        return self._detect_seasonal_anomaly(timestamp, value)
        
    def _detect_seasonal_anomaly(self, timestamp: float, value: float) -> Dict[str, any]:
        """Detect anomalies considering seasonal patterns"""
        # Get data from same time in previous periods
        current_hour = int((timestamp % 86400) / 3600)  # Hour of day
        seasonal_values = []
        
        for ts, val in self.historical_data:
            data_hour = int((ts % 86400) / 3600)
            if data_hour == current_hour:
                seasonal_values.append(val)
                
        if len(seasonal_values) < 3:
            return {'is_anomaly': False, 'reason': 'insufficient_seasonal_data'}
            
        # Calculate seasonal statistics
        seasonal_mean = np.mean(seasonal_values)
        seasonal_std = np.std(seasonal_values)
        
        if seasonal_std == 0:
            return {'is_anomaly': False, 'reason': 'no_seasonal_variation'}
            
        # Check for anomaly
        z_score = abs((value - seasonal_mean) / seasonal_std)
        is_anomaly = z_score > 2.5
        
        return {
            'is_anomaly': is_anomaly,
            'z_score': z_score,
            'seasonal_mean': seasonal_mean,
            'seasonal_std': seasonal_std,
            'reason': 'seasonal_analysis'
        }

# Machine Learning-based Anomaly Detection
class MLAnomalyDetector:
    def __init__(self, contamination: float = 0.1):
        self.contamination = contamination
        self.model = None
        self.is_trained = False
        self.training_data = []
        
    def add_training_data(self, features: List[float]):
        """Add data for training the model"""
        self.training_data.append(features)
        
    def train_model(self):
        """Train the anomaly detection model"""
        if len(self.training_data) < 50:
            raise ValueError("Need at least 50 training samples")
            
        try:
            from sklearn.ensemble import IsolationForest
            
            self.model = IsolationForest(
                contamination=self.contamination,
                random_state=42
            )
            
            X = np.array(self.training_data)
            self.model.fit(X)
            self.is_trained = True
            
        except ImportError:
            # Fallback to simple statistical method
            self._train_statistical_model()
            
    def _train_statistical_model(self):
        """Fallback statistical model"""
        X = np.array(self.training_data)
        self.mean = np.mean(X, axis=0)
        self.cov = np.cov(X.T)
        self.is_trained = True
        
    def predict_anomaly(self, features: List[float]) -> Dict[str, any]:
        """Predict if features represent an anomaly"""
        if not self.is_trained:
            return {'is_anomaly': False, 'reason': 'model_not_trained'}
            
        if self.model is not None:
            # Use Isolation Forest
            prediction = self.model.predict([features])[0]
            score = self.model.decision_function([features])[0]
            
            return {
                'is_anomaly': prediction == -1,
                'anomaly_score': score,
                'method': 'isolation_forest'
            }
        else:
            # Use statistical method (Mahalanobis distance)
            features_array = np.array(features)
            diff = features_array - self.mean
            
            try:
                inv_cov = np.linalg.inv(self.cov)
                mahal_dist = np.sqrt(diff.T @ inv_cov @ diff)
                
                # Use chi-square distribution for threshold
                threshold = stats.chi2.ppf(1 - self.contamination, len(features))
                
                return {
                    'is_anomaly': mahal_dist > threshold,
                    'mahalanobis_distance': mahal_dist,
                    'threshold': threshold,
                    'method': 'mahalanobis'
                }
                
            except np.linalg.LinAlgError:
                return {'is_anomaly': False, 'reason': 'singular_covariance_matrix'}

# Usage example
# Statistical detector
stat_detector = StatisticalAnomalyDetector(window_size=50, threshold=2.5)

# Simulate normal data with some anomalies
normal_data = np.random.normal(100, 10, 100)
anomalies = [150, 50, 200]  # Clear anomalies

for value in normal_data:
    is_anomaly = stat_detector.add_data_point(value)
    if is_anomaly:
        print(f"Anomaly detected: {value}")

# Time series detector
ts_detector = TimeSeriesAnomalyDetector(seasonal_period=24)

# Simulate hourly data with seasonal pattern
for hour in range(168):  # One week of hourly data
    timestamp = time.time() + hour * 3600
    # Simulate daily pattern with some noise
    base_value = 50 + 30 * np.sin(2 * np.pi * hour / 24) + np.random.normal(0, 5)
    
    result = ts_detector.add_data_point(timestamp, base_value)
    if result['is_anomaly']:
        print(f"Seasonal anomaly at hour {hour}: {base_value}")
```

## Root Cause Analysis

Root cause analysis helps identify the underlying causes of issues in distributed systems.

### Correlation Analysis

```python
import pandas as pd
import numpy as np
from typing import Dict, List, Tuple, Optional
from collections import defaultdict
import time

class CorrelationAnalyzer:
    def __init__(self, time_window: int = 300):  # 5 minutes
        self.time_window = time_window
        self.metrics_data = defaultdict(list)  # metric_name -> [(timestamp, value)]
        self.events_data = []  # [(timestamp, event_type, details)]
        
    def add_metric(self, metric_name: str, timestamp: float, value: float):
        """Add a metric data point"""
        self.metrics_data[metric_name].append((timestamp, value))
        
        # Keep only recent data
        cutoff_time = time.time() - self.time_window * 10  # Keep 10 windows
        self.metrics_data[metric_name] = [
            (ts, val) for ts, val in self.metrics_data[metric_name]
            if ts > cutoff_time
        ]
        
    def add_event(self, timestamp: float, event_type: str, details: Dict[str, any]):
        """Add an event (deployment, alert, etc.)"""
        self.events_data.append((timestamp, event_type, details))
        
        # Keep only recent events
        cutoff_time = time.time() - self.time_window * 10
        self.events_data = [
            (ts, et, det) for ts, et, det in self.events_data
            if ts > cutoff_time
        ]
        
    def find_correlations(self, target_metric: str, 
                         correlation_threshold: float = 0.7) -> List[Dict[str, any]]:
        """Find metrics correlated with target metric"""
        if target_metric not in self.metrics_data:
            return []
            
        target_data = self.metrics_data[target_metric]
        if len(target_data) < 10:
            return []
            
        correlations = []
        
        for metric_name, metric_data in self.metrics_data.items():
            if metric_name == target_metric or len(metric_data) < 10:
                continue
                
            correlation = self._calculate_correlation(target_data, metric_data)
            
            if abs(correlation) >= correlation_threshold:
                correlations.append({
                    'metric': metric_name,
                    'correlation': correlation,
                    'relationship': 'positive' if correlation > 0 else 'negative'
                })
                
        # Sort by absolute correlation value
        correlations.sort(key=lambda x: abs(x['correlation']), reverse=True)
        return correlations
        
    def _calculate_correlation(self, data1: List[Tuple[float, float]], 
                             data2: List[Tuple[float, float]]) -> float:
        """Calculate correlation between two time series"""
        # Align data points by timestamp (simplified approach)
        df1 = pd.DataFrame(data1, columns=['timestamp', 'value1'])
        df2 = pd.DataFrame(data2, columns=['timestamp', 'value2'])
        
        # Merge on timestamp with tolerance
        merged = pd.merge_asof(
            df1.sort_values('timestamp'),
            df2.sort_values('timestamp'),
            on='timestamp',
            tolerance=30  # 30 second tolerance
        )
        
        if len(merged) < 5:
            return 0.0
            
        return merged['value1'].corr(merged['value2'])
        
    def analyze_event_impact(self, event_timestamp: float, 
                           analysis_window: int = 600) -> Dict[str, any]:
        """Analyze metric changes around an event"""
        before_start = event_timestamp - analysis_window / 2
        after_end = event_timestamp + analysis_window / 2
        
        impact_analysis = {}
        
        for metric_name, metric_data in self.metrics_data.items():
            before_values = [
                value for ts, value in metric_data
                if before_start <= ts < event_timestamp
            ]
            
            after_values = [
                value for ts, value in metric_data
                if event_timestamp <= ts <= after_end
            ]
            
            if len(before_values) < 3 or len(after_values) < 3:
                continue
                
            before_mean = np.mean(before_values)
            after_mean = np.mean(after_values)
            
            # Calculate percentage change
            if before_mean != 0:
                change_percent = ((after_mean - before_mean) / before_mean) * 100
            else:
                change_percent = 0
                
            # Statistical significance test
            try:
                from scipy.stats import ttest_ind
                t_stat, p_value = ttest_ind(before_values, after_values)
                significant = p_value < 0.05
            except ImportError:
                significant = abs(change_percent) > 10  # Simple threshold
                p_value = None
                
            if abs(change_percent) > 5:  # Only include meaningful changes
                impact_analysis[metric_name] = {
                    'before_mean': before_mean,
                    'after_mean': after_mean,
                    'change_percent': change_percent,
                    'significant': significant,
                    'p_value': p_value
                }
                
        return impact_analysis

class RootCauseAnalyzer:
    def __init__(self):
        self.correlation_analyzer = CorrelationAnalyzer()
        self.dependency_graph = {}  # service -> [dependent_services]
        self.alert_history = []
        
    def add_service_dependency(self, service: str, dependencies: List[str]):
        """Add service dependency information"""
        self.dependency_graph[service] = dependencies
        
    def add_alert(self, timestamp: float, service: str, alert_type: str, 
                  severity: str, details: Dict[str, any]):
        """Add an alert to the history"""
        alert = {
            'timestamp': timestamp,
            'service': service,
            'alert_type': alert_type,
            'severity': severity,
            'details': details
        }
        self.alert_history.append(alert)
        
        # Keep only recent alerts
        cutoff_time = time.time() - 3600  # 1 hour
        self.alert_history = [
            alert for alert in self.alert_history
            if alert['timestamp'] > cutoff_time
        ]
        
    def analyze_alert_cascade(self, primary_alert_time: float, 
                            time_window: int = 300) -> Dict[str, any]:
        """Analyze if alerts are part of a cascade failure"""
        window_start = primary_alert_time
        window_end = primary_alert_time + time_window
        
        # Find alerts in the time window
        related_alerts = [
            alert for alert in self.alert_history
            if window_start <= alert['timestamp'] <= window_end
        ]
        
        if len(related_alerts) < 2:
            return {'cascade_detected': False}
            
        # Group alerts by service
        service_alerts = defaultdict(list)
        for alert in related_alerts:
            service_alerts[alert['service']].append(alert)
            
        # Analyze propagation pattern
        cascade_analysis = {
            'cascade_detected': True,
            'affected_services': list(service_alerts.keys()),
            'alert_timeline': sorted(related_alerts, key=lambda x: x['timestamp']),
            'propagation_analysis': self._analyze_propagation(service_alerts)
        }
        
        return cascade_analysis
        
    def _analyze_propagation(self, service_alerts: Dict[str, List[Dict]]) -> Dict[str, any]:
        """Analyze how alerts propagated through services"""
        propagation_paths = []
        
        # Find potential root cause services (those with dependencies)
        for service, dependencies in self.dependency_graph.items():
            if service in service_alerts:
                service_first_alert = min(service_alerts[service], 
                                        key=lambda x: x['timestamp'])
                
                # Check if dependent services also alerted
                dependent_alerts = []
                for dep_service in dependencies:
                    if dep_service in service_alerts:
                        dep_first_alert = min(service_alerts[dep_service],
                                            key=lambda x: x['timestamp'])
                        dependent_alerts.append({
                            'service': dep_service,
                            'first_alert_time': dep_first_alert['timestamp'],
                            'delay': dep_first_alert['timestamp'] - service_first_alert['timestamp']
                        })
                
                if dependent_alerts:
                    propagation_paths.append({
                        'root_service': service,
                        'root_alert_time': service_first_alert['timestamp'],
                        'affected_dependencies': dependent_alerts
                    })
                    
        return {
            'propagation_paths': propagation_paths,
            'likely_root_causes': self._identify_root_causes(propagation_paths)
        }
        
    def _identify_root_causes(self, propagation_paths: List[Dict]) -> List[str]:
        """Identify likely root cause services"""
        root_cause_scores = defaultdict(int)
        
        for path in propagation_paths:
            root_service = path['root_service']
            affected_count = len(path['affected_dependencies'])
            
            # Score based on number of affected dependencies and timing
            score = affected_count
            
            # Bonus for being early in the cascade
            avg_delay = np.mean([dep['delay'] for dep in path['affected_dependencies']])
            if avg_delay < 60:  # Quick propagation suggests strong causation
                score += 2
                
            root_cause_scores[root_service] += score
            
        # Return services sorted by likelihood of being root cause
        sorted_causes = sorted(root_cause_scores.items(), 
                             key=lambda x: x[1], reverse=True)
        
        return [service for service, score in sorted_causes]

# Usage example
rca = RootCauseAnalyzer()

# Set up service dependencies
rca.add_service_dependency('web-frontend', ['user-service', 'product-service'])
rca.add_service_dependency('user-service', ['database', 'cache'])
rca.add_service_dependency('product-service', ['database', 'search-service'])

# Simulate a cascade failure
base_time = time.time()

# Database starts having issues
rca.add_alert(base_time, 'database', 'high_latency', 'warning', 
              {'avg_query_time': 500})

# Services depending on database start alerting
rca.add_alert(base_time + 30, 'user-service', 'timeout_errors', 'critical',
              {'error_rate': 15})

rca.add_alert(base_time + 45, 'product-service', 'timeout_errors', 'critical',
              {'error_rate': 20})

# Frontend starts having issues
rca.add_alert(base_time + 60, 'web-frontend', 'high_error_rate', 'critical',
              {'error_rate': 25})

# Analyze the cascade
cascade_analysis = rca.analyze_alert_cascade(base_time, time_window=300)
print(f"Cascade detected: {cascade_analysis['cascade_detected']}")
print(f"Likely root causes: {cascade_analysis['propagation_analysis']['likely_root_causes']}")
```

## Conclusion

Observability in distributed systems requires a comprehensive approach combining metrics, logs, and traces with intelligent analysis capabilities. Key takeaways:

1. **Implement the Three Pillars**: Metrics, logs, and traces provide complementary views
2. **Use Structured Logging**: Makes logs searchable and analyzable
3. **Collect Meaningful Metrics**: Focus on business and system health indicators
4. **Implement Anomaly Detection**: Proactively identify issues before they impact users
5. **Build Root Cause Analysis**: Quickly identify and resolve the source of problems

The key is to build observability into your system from the beginning and continuously refine your monitoring and alerting based on operational experience.

## Next Steps

In the final section, we'll explore:
- **HLD Interview Problems**: Applying all these concepts in system design interviews