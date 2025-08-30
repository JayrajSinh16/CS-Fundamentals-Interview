# Data Warehousing

Data warehousing involves collecting, storing, and managing large volumes of data from multiple sources to support business intelligence, analytics, and reporting. Understanding OLAP vs OLTP systems is crucial for designing scalable data architecture.

## 🏗️ Data Warehouse Architecture

```mermaid
graph TB
    subgraph "Data Sources"
        OLTP1[(OLTP Database 1<br/>Orders)]
        OLTP2[(OLTP Database 2<br/>Users)]
        OLTP3[(OLTP Database 3<br/>Products)]
        EXT1[External APIs]
        FILES[Log Files]
        STREAM[Streaming Data]
    end
    
    subgraph "ETL Pipeline"
        ETL[ETL Process]
        EXTRACT[Extract]
        TRANSFORM[Transform]
        LOAD[Load]
        
        EXTRACT --> TRANSFORM
        TRANSFORM --> LOAD
    end
    
    subgraph "Data Warehouse"
        STAGE[Staging Area]
        DW[(Data Warehouse<br/>OLAP)]
        DM1[(Data Mart 1<br/>Sales)]
        DM2[(Data Mart 2<br/>Marketing)]
        DM3[(Data Mart 3<br/>Finance)]
    end
    
    subgraph "Analytics Layer"
        BI[Business Intelligence]
        DASH[Dashboards]
        REP[Reports]
        ML[Machine Learning]
        AD_HOC[Ad-hoc Queries]
    end
    
    OLTP1 --> ETL
    OLTP2 --> ETL
    OLTP3 --> ETL
    EXT1 --> ETL
    FILES --> ETL
    STREAM --> ETL
    
    ETL --> STAGE
    STAGE --> DW
    DW --> DM1
    DW --> DM2
    DW --> DM3
    
    DW --> BI
    DM1 --> DASH
    DM2 --> REP
    DM3 --> ML
    DW --> AD_HOC
    
    style DW fill:#ff9999
    style ETL fill:#99ff99
    style BI fill:#9999ff
```

## 🔧 OLTP vs OLAP Comparison

### OLTP (Online Transaction Processing)

```python
import asyncio
import asyncpg
import time
from typing import Dict, List, Optional, Any
from dataclasses import dataclass
from datetime import datetime, timedelta
import json

@dataclass
class TransactionMetrics:
    transaction_id: str
    start_time: float
    end_time: float
    operation_type: str
    affected_rows: int
    response_time_ms: float
    
    @property
    def duration_ms(self) -> float:
        return (self.end_time - self.start_time) * 1000

class OLTPDatabase:
    """OLTP database optimized for high-frequency transactions"""
    
    def __init__(self, connection_string: str, max_connections: int = 20):
        self.connection_string = connection_string
        self.max_connections = max_connections
        self.pool = None
        
        # OLTP characteristics
        self.transaction_log: List[TransactionMetrics] = []
        self.active_transactions = 0
        self.lock_manager = {}  # Table locks for ACID compliance
        
        # Performance metrics
        self.metrics = {
            'total_transactions': 0,
            'successful_transactions': 0,
            'failed_transactions': 0,
            'avg_response_time_ms': 0,
            'transactions_per_second': 0,
            'deadlocks': 0,
            'lock_waits': 0
        }
    
    async def initialize(self):
        """Initialize connection pool"""
        self.pool = await asyncpg.create_pool(
            self.connection_string,
            min_size=5,
            max_size=self.max_connections,
            max_queries=50000,
            max_inactive_connection_lifetime=300
        )
    
    async def execute_transaction(self, operations: List[Dict[str, Any]]) -> Dict[str, Any]:
        """Execute ACID transaction with multiple operations"""
        transaction_id = f"txn_{int(time.time() * 1000000)}"
        start_time = time.time()
        
        try:
            async with self.pool.acquire() as conn:
                async with conn.transaction():
                    self.active_transactions += 1
                    results = []
                    
                    for operation in operations:
                        if operation['type'] == 'INSERT':
                            result = await self._insert_record(conn, operation)
                        elif operation['type'] == 'UPDATE':
                            result = await self._update_record(conn, operation)
                        elif operation['type'] == 'DELETE':
                            result = await self._delete_record(conn, operation)
                        elif operation['type'] == 'SELECT':
                            result = await self._select_record(conn, operation)
                        else:
                            raise ValueError(f"Unknown operation type: {operation['type']}")
                        
                        results.append(result)
                    
                    end_time = time.time()
                    
                    # Log transaction metrics
                    metrics = TransactionMetrics(
                        transaction_id=transaction_id,
                        start_time=start_time,
                        end_time=end_time,
                        operation_type="TRANSACTION",
                        affected_rows=sum(r.get('affected_rows', 0) for r in results),
                        response_time_ms=(end_time - start_time) * 1000
                    )
                    
                    self.transaction_log.append(metrics)
                    self.active_transactions -= 1
                    
                    # Update metrics
                    self.metrics['total_transactions'] += 1
                    self.metrics['successful_transactions'] += 1
                    self._update_performance_metrics()
                    
                    return {
                        'transaction_id': transaction_id,
                        'status': 'success',
                        'results': results,
                        'duration_ms': metrics.duration_ms
                    }
        
        except Exception as e:
            self.active_transactions -= 1
            self.metrics['failed_transactions'] += 1
            
            return {
                'transaction_id': transaction_id,
                'status': 'error',
                'error': str(e),
                'duration_ms': (time.time() - start_time) * 1000
            }
    
    async def _insert_record(self, conn, operation: Dict) -> Dict:
        """Insert single record"""
        table = operation['table']
        data = operation['data']
        
        columns = list(data.keys())
        values = list(data.values())
        placeholders = [f'${i+1}' for i in range(len(values))]
        
        query = f"""
            INSERT INTO {table} ({', '.join(columns)})
            VALUES ({', '.join(placeholders)})
            RETURNING id
        """
        
        result = await conn.fetchrow(query, *values)
        
        return {
            'operation': 'INSERT',
            'table': table,
            'affected_rows': 1,
            'inserted_id': result['id'] if result else None
        }
    
    async def _update_record(self, conn, operation: Dict) -> Dict:
        """Update records"""
        table = operation['table']
        data = operation['data']
        conditions = operation['conditions']
        
        set_clauses = []
        values = []
        param_index = 1
        
        for column, value in data.items():
            set_clauses.append(f"{column} = ${param_index}")
            values.append(value)
            param_index += 1
        
        where_clauses = []
        for column, value in conditions.items():
            where_clauses.append(f"{column} = ${param_index}")
            values.append(value)
            param_index += 1
        
        query = f"""
            UPDATE {table}
            SET {', '.join(set_clauses)}
            WHERE {' AND '.join(where_clauses)}
        """
        
        result = await conn.execute(query, *values)
        affected_rows = int(result.split()[-1])
        
        return {
            'operation': 'UPDATE',
            'table': table,
            'affected_rows': affected_rows
        }
    
    async def _delete_record(self, conn, operation: Dict) -> Dict:
        """Delete records"""
        table = operation['table']
        conditions = operation['conditions']
        
        where_clauses = []
        values = []
        param_index = 1
        
        for column, value in conditions.items():
            where_clauses.append(f"{column} = ${param_index}")
            values.append(value)
            param_index += 1
        
        query = f"""
            DELETE FROM {table}
            WHERE {' AND '.join(where_clauses)}
        """
        
        result = await conn.execute(query, *values)
        affected_rows = int(result.split()[-1])
        
        return {
            'operation': 'DELETE',
            'table': table,
            'affected_rows': affected_rows
        }
    
    async def _select_record(self, conn, operation: Dict) -> Dict:
        """Select records"""
        table = operation['table']
        columns = operation.get('columns', ['*'])
        conditions = operation.get('conditions', {})
        limit = operation.get('limit', 100)
        
        column_list = ', '.join(columns)
        values = []
        param_index = 1
        
        query = f"SELECT {column_list} FROM {table}"
        
        if conditions:
            where_clauses = []
            for column, value in conditions.items():
                where_clauses.append(f"{column} = ${param_index}")
                values.append(value)
                param_index += 1
            
            query += f" WHERE {' AND '.join(where_clauses)}"
        
        query += f" LIMIT {limit}"
        
        rows = await conn.fetch(query, *values)
        
        return {
            'operation': 'SELECT',
            'table': table,
            'rows': [dict(row) for row in rows],
            'row_count': len(rows)
        }
    
    def _update_performance_metrics(self):
        """Update performance metrics"""
        if self.transaction_log:
            recent_transactions = self.transaction_log[-100:]  # Last 100 transactions
            
            # Calculate average response time
            total_time = sum(txn.duration_ms for txn in recent_transactions)
            self.metrics['avg_response_time_ms'] = total_time / len(recent_transactions)
            
            # Calculate TPS (transactions per second)
            if len(recent_transactions) >= 2:
                time_span = recent_transactions[-1].end_time - recent_transactions[0].start_time
                if time_span > 0:
                    self.metrics['transactions_per_second'] = len(recent_transactions) / time_span
    
    def get_oltp_metrics(self) -> Dict:
        """Get OLTP-specific metrics"""
        return {
            **self.metrics,
            'active_transactions': self.active_transactions,
            'total_logged_transactions': len(self.transaction_log),
            'characteristics': {
                'focus': 'High-frequency transactions',
                'operations': 'INSERT, UPDATE, DELETE',
                'data_freshness': 'Real-time',
                'query_complexity': 'Simple',
                'response_time': 'Milliseconds',
                'concurrency': 'High',
                'data_volume': 'Current operational data'
            }
        }
```

### OLAP (Online Analytical Processing)

```python
import pandas as pd
import numpy as np
from typing import Dict, List, Any, Optional, Tuple
from dataclasses import dataclass
import sqlite3
import time
from datetime import datetime, timedelta

@dataclass
class CubeDefinition:
    name: str
    fact_table: str
    dimensions: List[str]
    measures: List[str]
    aggregations: List[str]  # SUM, COUNT, AVG, MIN, MAX

@dataclass
class QueryProfile:
    query_id: str
    sql: str
    execution_time_ms: float
    rows_scanned: int
    rows_returned: int
    cache_hit: bool = False

class OLAPEngine:
    """OLAP engine optimized for analytical queries"""
    
    def __init__(self, data_warehouse_path: str):
        self.data_warehouse_path = data_warehouse_path
        self.connection = None
        
        # OLAP characteristics
        self.cubes: Dict[str, CubeDefinition] = {}
        self.query_cache: Dict[str, Any] = {}
        self.query_profiles: List[QueryProfile] = []
        self.materialized_views: Dict[str, str] = {}
        
        # Performance optimizations
        self.indexes = set()
        self.partitions: Dict[str, List[str]] = {}
        
        # Metrics
        self.metrics = {
            'total_queries': 0,
            'cache_hits': 0,
            'avg_query_time_ms': 0,
            'total_data_scanned_gb': 0,
            'materialized_view_hits': 0
        }
    
    def initialize(self):
        """Initialize OLAP engine"""
        self.connection = sqlite3.connect(self.data_warehouse_path)
        self.connection.row_factory = sqlite3.Row
        
        # Create sample fact and dimension tables
        self._create_sample_schema()
    
    def _create_sample_schema(self):
        """Create sample data warehouse schema"""
        
        # Fact table: Sales
        self.connection.execute("""
            CREATE TABLE IF NOT EXISTS fact_sales (
                sale_id INTEGER PRIMARY KEY,
                product_id INTEGER,
                customer_id INTEGER,
                date_id INTEGER,
                store_id INTEGER,
                quantity INTEGER,
                unit_price DECIMAL(10,2),
                total_amount DECIMAL(10,2),
                cost DECIMAL(10,2),
                profit DECIMAL(10,2)
            )
        """)
        
        # Dimension tables
        self.connection.execute("""
            CREATE TABLE IF NOT EXISTS dim_product (
                product_id INTEGER PRIMARY KEY,
                product_name TEXT,
                category TEXT,
                brand TEXT,
                price DECIMAL(10,2)
            )
        """)
        
        self.connection.execute("""
            CREATE TABLE IF NOT EXISTS dim_customer (
                customer_id INTEGER PRIMARY KEY,
                customer_name TEXT,
                age_group TEXT,
                city TEXT,
                state TEXT,
                country TEXT
            )
        """)
        
        self.connection.execute("""
            CREATE TABLE IF NOT EXISTS dim_date (
                date_id INTEGER PRIMARY KEY,
                date_value DATE,
                year INTEGER,
                quarter INTEGER,
                month INTEGER,
                week INTEGER,
                day_of_week INTEGER
            )
        """)
        
        self.connection.execute("""
            CREATE TABLE IF NOT EXISTS dim_store (
                store_id INTEGER PRIMARY KEY,
                store_name TEXT,
                city TEXT,
                state TEXT,
                region TEXT,
                store_type TEXT
            )
        """)
        
        self.connection.commit()
        
        # Create indexes for better performance
        self._create_performance_indexes()
    
    def _create_performance_indexes(self):
        """Create indexes for OLAP performance"""
        indexes = [
            "CREATE INDEX IF NOT EXISTS idx_sales_product ON fact_sales(product_id)",
            "CREATE INDEX IF NOT EXISTS idx_sales_customer ON fact_sales(customer_id)", 
            "CREATE INDEX IF NOT EXISTS idx_sales_date ON fact_sales(date_id)",
            "CREATE INDEX IF NOT EXISTS idx_sales_store ON fact_sales(store_id)",
            "CREATE INDEX IF NOT EXISTS idx_date_year_month ON dim_date(year, month)",
            "CREATE INDEX IF NOT EXISTS idx_product_category ON dim_product(category)",
            "CREATE INDEX IF NOT EXISTS idx_customer_location ON dim_customer(country, state, city)"
        ]
        
        for index_sql in indexes:
            self.connection.execute(index_sql)
            self.indexes.add(index_sql.split()[5])  # Extract index name
        
        self.connection.commit()
    
    def define_cube(self, cube: CubeDefinition):
        """Define OLAP cube"""
        self.cubes[cube.name] = cube
        
        # Create materialized view for cube
        self._create_materialized_view(cube)
    
    def _create_materialized_view(self, cube: CubeDefinition):
        """Create materialized view for cube performance"""
        
        # Build aggregation query
        dimension_columns = []
        for dim in cube.dimensions:
            if dim == 'product':
                dimension_columns.extend(['p.category', 'p.brand'])
            elif dim == 'date':
                dimension_columns.extend(['d.year', 'd.quarter', 'd.month'])
            elif dim == 'customer':
                dimension_columns.extend(['c.country', 'c.state', 'c.age_group'])
            elif dim == 'store':
                dimension_columns.extend(['s.region', 's.store_type'])
        
        measure_columns = []
        for measure in cube.measures:
            for agg in cube.aggregations:
                measure_columns.append(f"{agg}({measure}) as {agg.lower()}_{measure}")
        
        view_name = f"mv_{cube.name}"
        
        query = f"""
            CREATE VIEW IF NOT EXISTS {view_name} AS
            SELECT 
                {', '.join(dimension_columns)},
                {', '.join(measure_columns)}
            FROM {cube.fact_table} f
            LEFT JOIN dim_product p ON f.product_id = p.product_id
            LEFT JOIN dim_date d ON f.date_id = d.date_id
            LEFT JOIN dim_customer c ON f.customer_id = c.customer_id
            LEFT JOIN dim_store s ON f.store_id = s.store_id
            GROUP BY {', '.join(dimension_columns)}
        """
        
        self.connection.execute(query)
        self.materialized_views[cube.name] = view_name
        self.connection.commit()
    
    def execute_analytical_query(self, 
                                query_type: str,
                                dimensions: List[str],
                                measures: List[str],
                                filters: Optional[Dict] = None,
                                time_range: Optional[Tuple[str, str]] = None) -> Dict[str, Any]:
        """Execute analytical query"""
        
        query_id = f"query_{int(time.time() * 1000000)}"
        start_time = time.time()
        
        # Check cache first
        cache_key = self._generate_cache_key(query_type, dimensions, measures, filters, time_range)
        
        if cache_key in self.query_cache:
            self.metrics['cache_hits'] += 1
            
            profile = QueryProfile(
                query_id=query_id,
                sql="CACHE_HIT",
                execution_time_ms=1,
                rows_scanned=0,
                rows_returned=len(self.query_cache[cache_key]),
                cache_hit=True
            )
            
            self.query_profiles.append(profile)
            return self.query_cache[cache_key]
        
        # Build analytical query
        sql_query = self._build_analytical_query(query_type, dimensions, measures, filters, time_range)
        
        # Execute query
        cursor = self.connection.cursor()
        cursor.execute(sql_query)
        rows = cursor.fetchall()
        
        end_time = time.time()
        execution_time_ms = (end_time - start_time) * 1000
        
        # Convert to dictionary format
        result = {
            'query_type': query_type,
            'dimensions': dimensions,
            'measures': measures,
            'data': [dict(row) for row in rows],
            'row_count': len(rows),
            'execution_time_ms': execution_time_ms
        }
        
        # Cache result
        self.query_cache[cache_key] = result
        
        # Profile query
        profile = QueryProfile(
            query_id=query_id,
            sql=sql_query,
            execution_time_ms=execution_time_ms,
            rows_scanned=self._estimate_rows_scanned(sql_query),
            rows_returned=len(rows)
        )
        
        self.query_profiles.append(profile)
        
        # Update metrics
        self.metrics['total_queries'] += 1
        self._update_olap_metrics()
        
        return result
    
    def _build_analytical_query(self,
                               query_type: str,
                               dimensions: List[str],
                               measures: List[str],
                               filters: Optional[Dict] = None,
                               time_range: Optional[Tuple[str, str]] = None) -> str:
        """Build SQL query for analytics"""
        
        # Select clause
        select_parts = []
        
        # Add dimensions
        for dim in dimensions:
            if dim == 'product_category':
                select_parts.append('p.category')
            elif dim == 'product_brand':
                select_parts.append('p.brand')
            elif dim == 'customer_country':
                select_parts.append('c.country')
            elif dim == 'customer_age_group':
                select_parts.append('c.age_group')
            elif dim == 'date_year':
                select_parts.append('d.year')
            elif dim == 'date_month':
                select_parts.append('d.month')
            elif dim == 'store_region':
                select_parts.append('s.region')
        
        # Add measures
        for measure in measures:
            if query_type == 'sum':
                select_parts.append(f'SUM(f.{measure}) as total_{measure}')
            elif query_type == 'avg':
                select_parts.append(f'AVG(f.{measure}) as avg_{measure}')
            elif query_type == 'count':
                select_parts.append(f'COUNT(f.{measure}) as count_{measure}')
            elif query_type == 'min':
                select_parts.append(f'MIN(f.{measure}) as min_{measure}')
            elif query_type == 'max':
                select_parts.append(f'MAX(f.{measure}) as max_{measure}')
        
        # From clause with joins
        from_clause = """
            FROM fact_sales f
            LEFT JOIN dim_product p ON f.product_id = p.product_id
            LEFT JOIN dim_date d ON f.date_id = d.date_id  
            LEFT JOIN dim_customer c ON f.customer_id = c.customer_id
            LEFT JOIN dim_store s ON f.store_id = s.store_id
        """
        
        # Where clause
        where_conditions = []
        
        if time_range:
            where_conditions.append(f"d.date_value BETWEEN '{time_range[0]}' AND '{time_range[1]}'")
        
        if filters:
            for key, value in filters.items():
                if key == 'category':
                    where_conditions.append(f"p.category = '{value}'")
                elif key == 'country':
                    where_conditions.append(f"c.country = '{value}'")
                elif key == 'region':
                    where_conditions.append(f"s.region = '{value}'")
        
        where_clause = ""
        if where_conditions:
            where_clause = f"WHERE {' AND '.join(where_conditions)}"
        
        # Group by clause
        group_by_parts = [part for part in select_parts if 'SUM(' not in part and 'AVG(' not in part and 'COUNT(' not in part and 'MIN(' not in part and 'MAX(' not in part]
        group_by_clause = ""
        if group_by_parts:
            group_by_clause = f"GROUP BY {', '.join(group_by_parts)}"
        
        # Build complete query
        query = f"""
            SELECT {', '.join(select_parts)}
            {from_clause}
            {where_clause}
            {group_by_clause}
            ORDER BY {select_parts[0] if select_parts else '1'}
        """
        
        return query
    
    def drill_down(self, base_result: Dict, drill_dimension: str) -> Dict[str, Any]:
        """Drill down into more detailed data"""
        
        # Add more granular dimension
        new_dimensions = base_result['dimensions'] + [drill_dimension]
        
        return self.execute_analytical_query(
            query_type='sum',
            dimensions=new_dimensions,
            measures=base_result['measures']
        )
    
    def roll_up(self, base_result: Dict, remove_dimension: str) -> Dict[str, Any]:
        """Roll up to higher level aggregation"""
        
        # Remove specified dimension
        new_dimensions = [dim for dim in base_result['dimensions'] if dim != remove_dimension]
        
        return self.execute_analytical_query(
            query_type='sum',
            dimensions=new_dimensions,
            measures=base_result['measures']
        )
    
    def slice_and_dice(self, 
                      dimensions: List[str],
                      measures: List[str],
                      slice_filters: Dict[str, str]) -> Dict[str, Any]:
        """Slice and dice analysis"""
        
        return self.execute_analytical_query(
            query_type='sum',
            dimensions=dimensions,
            measures=measures,
            filters=slice_filters
        )
    
    def _generate_cache_key(self, query_type: str, dimensions: List[str], 
                           measures: List[str], filters: Optional[Dict], 
                           time_range: Optional[Tuple[str, str]]) -> str:
        """Generate cache key for query"""
        import hashlib
        
        key_parts = [
            query_type,
            '|'.join(sorted(dimensions)),
            '|'.join(sorted(measures)),
            str(sorted(filters.items()) if filters else ''),
            str(time_range) if time_range else ''
        ]
        
        key_string = '|'.join(key_parts)
        return hashlib.md5(key_string.encode()).hexdigest()
    
    def _estimate_rows_scanned(self, sql_query: str) -> int:
        """Estimate rows scanned (simplified)"""
        # In real implementation, would use EXPLAIN QUERY PLAN
        if 'fact_sales' in sql_query:
            return 1000000  # Assume 1M rows in fact table
        return 10000
    
    def _update_olap_metrics(self):
        """Update OLAP metrics"""
        if self.query_profiles:
            recent_queries = self.query_profiles[-100:]
            total_time = sum(q.execution_time_ms for q in recent_queries)
            self.metrics['avg_query_time_ms'] = total_time / len(recent_queries)
            
            total_scanned = sum(q.rows_scanned for q in recent_queries)
            self.metrics['total_data_scanned_gb'] = total_scanned / (1024 * 1024 * 1024)
    
    def get_olap_metrics(self) -> Dict:
        """Get OLAP-specific metrics"""
        return {
            **self.metrics,
            'active_cubes': len(self.cubes),
            'materialized_views': len(self.materialized_views),
            'cached_queries': len(self.query_cache),
            'characteristics': {
                'focus': 'Complex analytical queries',
                'operations': 'SELECT with aggregations',
                'data_freshness': 'Batch updated (hours/daily)',
                'query_complexity': 'Complex with joins and aggregations',
                'response_time': 'Seconds to minutes',
                'concurrency': 'Low to medium',
                'data_volume': 'Historical data (years)'
            }
        }
```

### 3. Modern Data Warehouse Implementation

```python
import asyncio
import aiofiles
import json
from pathlib import Path
from typing import Dict, List, Any, Optional
from dataclasses import dataclass, asdict
from datetime import datetime, timedelta
import pandas as pd

@dataclass
class DataWarehouseConfig:
    name: str
    storage_type: str  # "columnar", "row", "hybrid"
    compression: str  # "snappy", "gzip", "lz4"
    partitioning_strategy: str  # "time", "hash", "range"
    replication_factor: int = 3
    backup_retention_days: int = 30

class ModernDataWarehouse:
    """Modern cloud-native data warehouse"""
    
    def __init__(self, config: DataWarehouseConfig, base_path: str = "/data/warehouse"):
        self.config = config
        self.base_path = Path(base_path)
        
        # Storage layers
        self.cold_storage_path = self.base_path / "cold"
        self.warm_storage_path = self.base_path / "warm" 
        self.hot_storage_path = self.base_path / "hot"
        
        # Create directories
        for path in [self.cold_storage_path, self.warm_storage_path, self.hot_storage_path]:
            path.mkdir(parents=True, exist_ok=True)
        
        # Metadata management
        self.table_metadata: Dict[str, Dict] = {}
        self.partition_metadata: Dict[str, List[Dict]] = {}
        
        # Query engine
        self.query_engine = None
        self.statistics_collector = StatisticsCollector()
        
        # Performance optimizations
        self.bloom_filters: Dict[str, Any] = {}
        self.column_statistics: Dict[str, Dict] = {}
    
    async def create_table(self, 
                          table_name: str,
                          schema: Dict[str, str],
                          partition_columns: List[str] = None,
                          storage_tier: str = "hot") -> bool:
        """Create table in data warehouse"""
        
        if partition_columns is None:
            partition_columns = []
        
        table_meta = {
            'name': table_name,
            'schema': schema,
            'partition_columns': partition_columns,
            'storage_tier': storage_tier,
            'created_at': datetime.now().isoformat(),
            'row_count': 0,
            'size_bytes': 0,
            'last_updated': datetime.now().isoformat()
        }
        
        self.table_metadata[table_name] = table_meta
        
        # Create table directory structure
        table_path = self._get_storage_path(storage_tier) / table_name
        table_path.mkdir(parents=True, exist_ok=True)
        
        # Save metadata
        await self._save_table_metadata(table_name)
        
        return True
    
    async def insert_data(self, 
                         table_name: str, 
                         data: List[Dict[str, Any]],
                         partition_value: Optional[str] = None) -> bool:
        """Insert data into table"""
        
        if table_name not in self.table_metadata:
            raise ValueError(f"Table {table_name} does not exist")
        
        table_meta = self.table_metadata[table_name]
        
        # Determine partition
        if table_meta['partition_columns']:
            if not partition_value:
                # Auto-generate partition based on first partition column
                first_partition_col = table_meta['partition_columns'][0]
                if first_partition_col in data[0]:
                    partition_value = str(data[0][first_partition_col])
                else:
                    partition_value = "default"
        else:
            partition_value = "default"
        
        # Get or create partition
        partition_path = await self._get_or_create_partition(table_name, partition_value)
        
        # Save data in columnar format
        await self._save_columnar_data(partition_path, data)
        
        # Update metadata
        table_meta['row_count'] += len(data)
        table_meta['last_updated'] = datetime.now().isoformat()
        
        await self._save_table_metadata(table_name)
        
        return True
    
    async def query_data(self, 
                        sql_query: str,
                        optimize: bool = True) -> List[Dict[str, Any]]:
        """Execute query on data warehouse"""
        
        # Parse query (simplified)
        query_plan = await self._parse_and_optimize_query(sql_query, optimize)
        
        # Execute query plan
        results = await self._execute_query_plan(query_plan)
        
        return results
    
    async def _get_or_create_partition(self, table_name: str, partition_value: str) -> Path:
        """Get or create partition directory"""
        
        table_path = self._get_storage_path(self.table_metadata[table_name]['storage_tier']) / table_name
        partition_path = table_path / f"partition_{partition_value}"
        
        if not partition_path.exists():
            partition_path.mkdir(parents=True, exist_ok=True)
            
            # Add to partition metadata
            if table_name not in self.partition_metadata:
                self.partition_metadata[table_name] = []
            
            self.partition_metadata[table_name].append({
                'partition_value': partition_value,
                'path': str(partition_path),
                'created_at': datetime.now().isoformat(),
                'row_count': 0,
                'size_bytes': 0
            })
        
        return partition_path
    
    async def _save_columnar_data(self, partition_path: Path, data: List[Dict[str, Any]]):
        """Save data in columnar format"""
        
        if not data:
            return
        
        # Convert to pandas DataFrame for columnar operations
        df = pd.DataFrame(data)
        
        # Save each column separately (columnar storage)
        for column in df.columns:
            column_file = partition_path / f"{column}.parquet"
            
            # Save column data
            df[[column]].to_parquet(column_file, compression=self.config.compression)
        
        # Save row metadata
        metadata_file = partition_path / "metadata.json"
        metadata = {
            'row_count': len(data),
            'columns': list(df.columns),
            'created_at': datetime.now().isoformat(),
            'compression': self.config.compression
        }
        
        async with aiofiles.open(metadata_file, 'w') as f:
            await f.write(json.dumps(metadata, indent=2))
    
    async def _parse_and_optimize_query(self, sql_query: str, optimize: bool) -> Dict:
        """Parse and optimize SQL query"""
        
        # Simplified query parsing
        query_lower = sql_query.lower().strip()
        
        if query_lower.startswith('select'):
            # Extract components (very simplified)
            parts = sql_query.split()
            
            query_plan = {
                'type': 'SELECT',
                'tables': [],
                'columns': ['*'],
                'conditions': [],
                'aggregations': [],
                'optimizations': []
            }
            
            # Find table names (simplified)
            from_index = -1
            for i, part in enumerate(parts):
                if part.lower() == 'from':
                    from_index = i
                    break
            
            if from_index != -1 and from_index + 1 < len(parts):
                query_plan['tables'] = [parts[from_index + 1].rstrip(',')]
            
            # Add optimizations if enabled
            if optimize:
                query_plan['optimizations'] = [
                    'predicate_pushdown',
                    'column_pruning',
                    'partition_elimination'
                ]
            
            return query_plan
        
        raise ValueError(f"Unsupported query type: {sql_query}")
    
    async def _execute_query_plan(self, query_plan: Dict) -> List[Dict[str, Any]]:
        """Execute query plan"""
        
        if query_plan['type'] == 'SELECT':
            results = []
            
            for table_name in query_plan['tables']:
                if table_name in self.table_metadata:
                    table_data = await self._read_table_data(table_name)
                    results.extend(table_data)
            
            return results
        
        return []
    
    async def _read_table_data(self, table_name: str) -> List[Dict[str, Any]]:
        """Read data from table"""
        
        table_meta = self.table_metadata[table_name]
        table_path = self._get_storage_path(table_meta['storage_tier']) / table_name
        
        all_data = []
        
        # Read from all partitions
        for partition_dir in table_path.iterdir():
            if partition_dir.is_dir() and partition_dir.name.startswith('partition_'):
                partition_data = await self._read_partition_data(partition_dir)
                all_data.extend(partition_data)
        
        return all_data
    
    async def _read_partition_data(self, partition_path: Path) -> List[Dict[str, Any]]:
        """Read data from partition"""
        
        # Read metadata
        metadata_file = partition_path / "metadata.json"
        if not metadata_file.exists():
            return []
        
        async with aiofiles.open(metadata_file, 'r') as f:
            metadata = json.loads(await f.read())
        
        # Read columnar data
        data_frames = {}
        
        for column in metadata['columns']:
            column_file = partition_path / f"{column}.parquet"
            if column_file.exists():
                df = pd.read_parquet(column_file)
                data_frames[column] = df[column].tolist()
        
        # Reconstruct rows
        if data_frames:
            row_count = len(next(iter(data_frames.values())))
            rows = []
            
            for i in range(row_count):
                row = {col: data_frames[col][i] for col in data_frames}
                rows.append(row)
            
            return rows
        
        return []
    
    def _get_storage_path(self, storage_tier: str) -> Path:
        """Get storage path for tier"""
        if storage_tier == "hot":
            return self.hot_storage_path
        elif storage_tier == "warm":
            return self.warm_storage_path
        elif storage_tier == "cold":
            return self.cold_storage_path
        else:
            return self.hot_storage_path
    
    async def _save_table_metadata(self, table_name: str):
        """Save table metadata"""
        table_meta = self.table_metadata[table_name]
        metadata_file = self._get_storage_path(table_meta['storage_tier']) / table_name / "table_metadata.json"
        
        async with aiofiles.open(metadata_file, 'w') as f:
            await f.write(json.dumps(table_meta, indent=2, default=str))
    
    def get_warehouse_stats(self) -> Dict:
        """Get data warehouse statistics"""
        
        total_tables = len(self.table_metadata)
        total_rows = sum(meta['row_count'] for meta in self.table_metadata.values())
        total_size = sum(meta['size_bytes'] for meta in self.table_metadata.values())
        
        return {
            'config': asdict(self.config),
            'total_tables': total_tables,
            'total_rows': total_rows,
            'total_size_gb': total_size / (1024**3),
            'tables': {name: {
                'row_count': meta['row_count'],
                'size_bytes': meta['size_bytes'],
                'storage_tier': meta['storage_tier']
            } for name, meta in self.table_metadata.items()},
            'storage_distribution': {
                'hot': len([m for m in self.table_metadata.values() if m['storage_tier'] == 'hot']),
                'warm': len([m for m in self.table_metadata.values() if m['storage_tier'] == 'warm']),
                'cold': len([m for m in self.table_metadata.values() if m['storage_tier'] == 'cold'])
            }
        }

class StatisticsCollector:
    """Collect and maintain table statistics for optimization"""
    
    def __init__(self):
        self.table_stats: Dict[str, Dict] = {}
        self.column_stats: Dict[str, Dict[str, Dict]] = {}
    
    def collect_table_stats(self, table_name: str, data: List[Dict[str, Any]]):
        """Collect statistics for table"""
        
        if not data:
            return
        
        row_count = len(data)
        columns = list(data[0].keys())
        
        # Basic table statistics
        self.table_stats[table_name] = {
            'row_count': row_count,
            'column_count': len(columns),
            'avg_row_size': self._estimate_avg_row_size(data),
            'last_analyzed': datetime.now().isoformat()
        }
        
        # Column statistics
        self.column_stats[table_name] = {}
        
        for column in columns:
            values = [row.get(column) for row in data if row.get(column) is not None]
            
            if values:
                self.column_stats[table_name][column] = {
                    'distinct_count': len(set(values)),
                    'null_count': row_count - len(values),
                    'min_value': min(values) if values else None,
                    'max_value': max(values) if values else None,
                    'data_type': type(values[0]).__name__ if values else 'unknown'
                }
    
    def _estimate_avg_row_size(self, data: List[Dict[str, Any]]) -> float:
        """Estimate average row size in bytes"""
        if not data:
            return 0
        
        sample_size = min(100, len(data))
        total_size = 0
        
        for i in range(sample_size):
            row_size = sum(len(str(v).encode('utf-8')) for v in data[i].values())
            total_size += row_size
        
        return total_size / sample_size

# Demo usage
async def demo_data_warehouse():
    """Demonstrate data warehouse functionality"""
    
    print("=== Data Warehousing Demo ===")
    
    # 1. OLTP Database Demo
    print("\n1. OLTP Database:")
    
    # Note: Would need actual PostgreSQL connection for real demo
    # oltp = OLTPDatabase("postgresql://user:pass@localhost/oltp_db")
    # await oltp.initialize()
    
    print("OLTP Characteristics:")
    print("- High-frequency transactions")
    print("- ACID compliance")
    print("- Real-time data updates")
    print("- Simple queries with fast response")
    
    # 2. OLAP Engine Demo
    print("\n2. OLAP Engine:")
    
    olap = OLAPEngine("/tmp/datawarehouse.db")
    olap.initialize()
    
    # Define sales cube
    sales_cube = CubeDefinition(
        name="sales_analysis",
        fact_table="fact_sales",
        dimensions=["product", "date", "customer", "store"],
        measures=["quantity", "total_amount", "profit"],
        aggregations=["SUM", "AVG", "COUNT"]
    )
    
    olap.define_cube(sales_cube)
    
    # Example analytical query
    result = olap.execute_analytical_query(
        query_type="sum",
        dimensions=["product_category", "date_year"],
        measures=["total_amount", "profit"],
        filters={"category": "Electronics"},
        time_range=("2023-01-01", "2023-12-31")
    )
    
    print(f"Analytical query result: {result['row_count']} rows in {result['execution_time_ms']:.2f}ms")
    
    # 3. Modern Data Warehouse Demo
    print("\n3. Modern Data Warehouse:")
    
    config = DataWarehouseConfig(
        name="sales_warehouse",
        storage_type="columnar",
        compression="snappy",
        partitioning_strategy="time",
        replication_factor=3
    )
    
    warehouse = ModernDataWarehouse(config, "/tmp/modern_warehouse")
    
    # Create table
    await warehouse.create_table(
        "sales_fact",
        {
            "sale_id": "INTEGER",
            "product_id": "INTEGER", 
            "customer_id": "INTEGER",
            "sale_date": "DATE",
            "amount": "DECIMAL"
        },
        partition_columns=["sale_date"],
        storage_tier="hot"
    )
    
    # Insert sample data
    sample_data = [
        {"sale_id": 1, "product_id": 101, "customer_id": 1001, "sale_date": "2023-01-15", "amount": 99.99},
        {"sale_id": 2, "product_id": 102, "customer_id": 1002, "sale_date": "2023-01-16", "amount": 149.99},
        {"sale_id": 3, "product_id": 103, "customer_id": 1003, "sale_date": "2023-01-17", "amount": 79.99}
    ]
    
    await warehouse.insert_data("sales_fact", sample_data, "2023-01")
    
    # Query data
    results = await warehouse.query_data("SELECT * FROM sales_fact")
    print(f"Query returned {len(results)} rows")
    
    # 4. Compare OLTP vs OLAP
    print("\n4. OLTP vs OLAP Comparison:")
    
    comparison = {
        "Aspect": ["Purpose", "Operations", "Data Volume", "Query Complexity", "Response Time", "Users"],
        "OLTP": [
            "Transaction processing",
            "INSERT, UPDATE, DELETE", 
            "Current data",
            "Simple",
            "Milliseconds",
            "Many concurrent users"
        ],
        "OLAP": [
            "Analytical processing",
            "Complex SELECT",
            "Historical data",
            "Complex aggregations",
            "Seconds to minutes", 
            "Fewer analytical users"
        ]
    }
    
    for i, aspect in enumerate(comparison["Aspect"]):
        print(f"{aspect:15} | OLTP: {comparison['OLTP'][i]:25} | OLAP: {comparison['OLAP'][i]}")
    
    # 5. Statistics
    print("\n5. Data Warehouse Statistics:")
    warehouse_stats = warehouse.get_warehouse_stats()
    print(f"Total tables: {warehouse_stats['total_tables']}")
    print(f"Total rows: {warehouse_stats['total_rows']}")
    print(f"Storage distribution: {warehouse_stats['storage_distribution']}")

if __name__ == "__main__":
    asyncio.run(demo_data_warehouse())
```

---

**Key Takeaway**: Data warehousing bridges the gap between operational OLTP systems and analytical OLAP requirements. OLTP systems excel at high-frequency transactions with ACID guarantees, while OLAP systems are optimized for complex analytical queries on large historical datasets. Modern data warehouses combine columnar storage, intelligent partitioning, and cloud-native architectures to provide both performance and scalability for business intelligence and analytics.
