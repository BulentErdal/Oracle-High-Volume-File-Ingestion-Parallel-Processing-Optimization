![High Volume Architecture](architecture.png)

 <img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/71f855e0-630c-43ea-bf01-f8d4c274a7cb" /> 
 
 ### Business Context

A financial trading system was receiving approximately 17 million order book records as flat text files from an external market data feed.
These records needed to be ingested into the company’s Oracle database, processed, and prepared for downstream ordering, reporting, and reconciliation workflows.
The original ingestion and processing pipeline required approximately 1.5–2 hours to complete, creating significant pressure on batch windows and delaying critical business reporting.


 ### Technical Environment

Oracle Database (Enterprise scale)
High-volume flat file ingestion
Single target transactional table
Later redesigned with range-based partitioning
Heavy batch workload with concurrent processing requirements


 ### Performance Bottleneck Analysis

Detailed investigation revealed multiple architectural inefficiencies:
  File parsing and insert operations were handled row-by-row in PL/SQL
  Excessive SQL ↔ PL/SQL context switching
  No parallelism during ingestion stage
  Sequential ordering logic increased processing latency
  High redo / undo generation during bulk inserts
  Synchronous downstream processing increased total execution time
  

 ###  Optimization Strategy

To address these issues, the ingestion pipeline was fully redesigned:
  Replaced PL/SQL file parsing with SQL Loader Direct Path Load
  Introduced a staging architecture for decoupled ingestion and transformation
  Redesigned the target table using range partitioning to improve insert and query performance
  Implemented parallel batch processing using DBMS_JOB (up to 40 concurrent workers)
  Workload distributed using range-based data segmentation
  Refactored ordering and enrichment logic using analytic window functions
  Reduced procedural loops and migrated to set-based SQL operations
  Optimized commit strategy and reduced logging overhead
  Improved indexing and execution plan stability

  ### Parallel Processing Design

The workload was distributed across up to 40 DBMS_JOB workers using range-based segmentation of the staging dataset.  
Each worker processed a deterministic subset of records to avoid contention and maximize throughput.

Direct-path INSERT with APPEND hints combined with controlled commit batching ensured minimal redo overhead and optimal load performance.
🚀 Bu Repo’nun Etkisi (Gerçek)

 ### Results

File ingestion time reduced from ~40 minutes to ~3 minutes
End-to-end pipeline reduced from ~1.5–2 hours to ~10 minutes
Massive reduction in CPU utilization and I/O contention
Stable and predictable batch execution
Improved scalability for increasing market data volumes


 ### Key Takeaways
Bulk ingestion architecture redesign can deliver order-of-magnitude performance gains
Direct Path Load + Partitioning + Parallel Processing is a powerful combination
Analytic functions significantly outperform cursor-based sequential logic
Controlled parallelism must align with operational constraints (job limits, system capacity)

###  Architecture Diagram

<img width="306" height="632" alt="tum_emir drawio" src="https://github.com/user-attachments/assets/f8f919a9-28a1-47ad-9d00-e9d9e502376c" />


