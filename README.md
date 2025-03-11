# Order Book Snapshot Query Processing System Design Documentation

## Overview
The Order Book Snapshot Query Processing System is designed to efficiently handle high-frequency financial data. This system processes raw market data, maintains an up-to-date order book, periodically generates time-series snapshots, and supports optimized querying for historical data. This document details the design principles, architecture, algorithms, and future improvements.

---

## Functional Requirements

1. **Time-Series Data Storage**:
   - Persistently store order book snapshots on disk.
   - Ensure efficient retrieval for specified time ranges.

2. **Query Processing Engine**:
   - Retrieve snapshots based on time range, symbols, and specific fields.
   - Support single and multiple symbol queries.
   - Handle both full-field and selective-field queries.

3. **Order Data Handling**:
   - Process `NEW`, `CANCEL`, and `TRADE` orders.
   - Maintain an updated order book in memory.
   - Generate periodic snapshots reflecting the current state.

4. **Data Integrity**:
   - Ensure consistency and correctness of order book state and snapshots.

5. **Testing Framework**:
   - Provide automated unit and integration tests to validate core functionalities.

---

## Non-Functional Requirements

1. **Performance**:
   - Minimize query response time and optimize storage operations.
   - Efficiently handle high-frequency data streams.

2. **Scalability**:
   - Support increasing volumes of order data.

3. **Reliability**:
   - Ensure robustness against unexpected inputs or data corruption.

---

## System Architecture

### Components

1. **Order Processor**:
   - Parses raw order data (log files).
   - Validates and processes orders (e.g., `NEW`, `CANCEL`, `TRADE`).
   - Updates the in-memory order book.

2. **Order Book Manager**:
   - Manages multiple `OrderBook` objects, one for each symbol.

3. **Snapshot Manager**:
   - Periodically generates and stores snapshots to disk.
   - Maintains an index for efficient snapshot retrieval.

4. **Query Processor**:
   - Handles queries for time ranges, symbols, and fields.
   - Retrieves relevant snapshots and processes requested data.

5. **Data Storage**:
   - Stores raw market data and snapshots persistently.
   - Provides indexing for fast lookup.

6. **Testing Framework**:
   - Ensures system integrity and performance through automated tests.

---

## Data Structures

### Order
```cpp
class Order {
public:
    epoch_t epoch_;        // Timestamp of the order
    order_id_t order_id_;  // Unique identifier
    std::string symbol_;   // Tradable instrument symbol
    std::string side_;     // "BUY" or "SELL"
    std::string category_; // "NEW", "CANCEL", "TRADE"
    double price_;         // Order price
    int quantity_;         // Order quantity
};
```

### Order Book
```cpp
class OrderBook {
public:
    std::map<double, int, std::greater<double>> bids_; // Descending sorted bid levels
    std::map<double, int> asks_;                       // Ascending sorted ask levels
};
```

### Snapshot
```cpp
class Snapshot {
public:
    epoch_t epoch_;
    std::string symbol_;
    std::map<double, int> bids_; // Top 5 bid levels
    std::map<double, int> asks_; // Top 5 ask levels
    double last_trade_price_;
    int last_trade_quantity_;
};
```

---

## Algorithms

### Order Book Update
1. Parse raw order data into `Order` objects.
2. Validate the order (e.g., `CANCEL` must reference an existing order).
3. Update the `OrderBook`:
   - **NEW**: Add or update the bid/ask level.
   - **CANCEL**: Remove or adjust the bid/ask level.
   - **TRADE**: Adjust quantities or remove levels if fully traded.

### Snapshot Generation
1. Periodically capture the `OrderBook` state.
2. Serialize and store snapshots to disk.
3. Update indices for quick lookup.

### Query Processing
1. Identify relevant snapshots for the query's time range.
2. Filter data by symbol and fields.
3. Reconstruct additional states if required (e.g., extended ranges).
4. Return results in the specified format.

---

## Optimization Strategies

1. **Caching**:
   - Cache frequently accessed snapshots for quick retrieval.

2. **Indexing**:
   - Use time-based and symbol-based indices to reduce disk scans.

3. **Multithreading**:
   - Parallelize log processing, snapshot generation, and query handling.

---

## Testing Strategy

1. **Unit Tests**:
   - Validate `Order`, `OrderBook`, and `Snapshot` operations.
   - Test parsing, validation, and order book updates.

2. **Integration Tests**:
   - Simulate end-to-end workflows, including log processing, snapshot generation, and queries.

3. **Edge Case Testing**:
   - Handle scenarios such as invalid orders, overlapping time ranges, and large datasets.

---

## Future Enhancements

1. **Query Input Stabilization**:
   - Enhance the stability of the query input system in multi-threaded environments.
   - Current implementation allows real-time input handling but shows potential instability under concurrent usage.
   - Introduce thread-safe mechanisms (e.g., refined mutex locking, thread-local storage) to ensure consistent and reliable query processing.

2. **Microservices Architecture**:
   - Modularize components (e.g., Order Processor, Snapshot Manager) into independent services.

3. **Advanced Indexing**:
   - Use B-Trees for efficient range queries and data retrieval.

4. **Data Compression**:
   - Implement compression for stored snapshots to save disk space.

5. **Improved Query Optimization**:
   - Introduce query plan caching to accelerate repeated queries with identical parameters.

6. **Real-Time Snapshot Updates**:
   - Enable incremental updates to snapshots in real-time to reduce processing overhead during query retrieval.

7. **Enhanced Error Handling**:
   - Develop more robust logging and error recovery mechanisms, particularly for edge cases like corrupted log files or disk I/O failures.

8. **Preloading and Prefetching Mechanisms**:
   - Predictively load snapshots or data into memory for frequently queried time ranges and symbols to improve responsiveness.

9. **Base and Differential Snapshot Separation**:
   - Separate base snapshots (full order book state) from differential snapshots (incremental changes).
   - This approach reduces storage overhead by capturing only incremental changes after a base snapshot.
   - Queries can reconstruct the order book by applying differential snapshots to the nearest base snapshot.

10. **Batch Processing for Logs**:
    - Implement batch processing to read and process multiple log lines at once.
    - This reduces I/O overhead and improves the efficiency of log handling.

11. **Memory Management Component**:
    - Introduce a dedicated memory management component.
    - Monitor memory usage across components and enforce thresholds to trigger cleanup or swapping mechanisms.
    - Ensure stable performance under high data load by reclaiming memory from less critical caches or data structures.

---

## Conclusion
This design ensures that the system meets the requirements of high-frequency financial data processing, offering scalability, performance, and reliability. Future enhancements will focus on modularity, advanced indexing, and data optimization.
