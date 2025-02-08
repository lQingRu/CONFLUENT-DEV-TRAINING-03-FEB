Some questions that I asked during class:

### 1. Is it considered a best practice to always include a key when sending records in Kafka?
- If not, what are some good use cases for using keys versus not using keys?
- Additionally, if we do use keys, does this require careful key design to ensure even distribution across partitions, given that Kafka handles partitioning based on keys?

 **Instructor's**
 - Partitioner decides this message from producer where should be landing (which partition)
    - Default partitioner: `hash(key) % no.of partitions`
        - Ensures messages with the same key will land to the same partition
- To prevent partition skew when using own keys, may need to implement custom partitioning strategy to promote balanced distribution (will visit this in the later modules)
    - NOTE: Should make sure same parameters go to the same partition
 
### 2. How can we ensure that messages from Partition 1 are fully processed before messages from Partition 2, given that different consumers in the same consumer group may process at different speeds?
- OR actually the designing of partitions should have already considered this?

**Instructor's**
- Will need to monitor `lag` then adjust throughput of producers / scale consumers accordingly

### 3. Does the limitation of range partition assignment mean it's not as extensible, especially when there's a requirement for the same number of partitions between topics? What happens if a topic expands significantly, requiring more partitions?
**Instructor's**
- Yes
- Need to use range partition assignment cautiously, only in specific scenarios where we are certain topics are co-related

### 4. Would there be scenarios where we want to allow reading from followers over waiting for a leader re-election process?
**Instructor's**
- During leader re-election, cannot have consumption (think he misunderstood my question)

**Mine** 
- Could be cases where we still want to allow consumption to take place, without the priority of in-order messages

### 5. When evolving a schema in Kafka, is it better to ensure backward compatibility by adding default values to new fields (there can be cases where we may not want to simply just add default values?), or should I create a new schema version to avoid potential issues with existing messages in the topic?
**Mine**
- Depends if the changes are drastic that we cannot ensure compatibility easily
- Also depends if the new schema has the same purpose (i.e. logical grouping)

### 6. Why is there a potential incompatibility when schema evolves?
- Isn’t a message tagged with schema ID (which identifies specific version) so consumer should refer the same schema that was first tagged in the message by the producer?

**Mine**
- That is because:
   - See more here https://forum.confluent.io/t/when-using-the-schema-registry-why-do-we-still-need-old-reader-compatibility-checking/9912/4
   - Schema acts as a contract between producers and consumers
   - There might be a strict typing in consumer code where it is expecting certain type of schema for the message in the topic
       
        ![image](https://github.com/user-attachments/assets/a115cce1-70ff-4742-9a24-d5860bf6c813)
        
   - Also, seems like no easy way to see the message sent by producer is using what schema - will need to do some specific buffer read
       - OR to take in generic record to see the schema from the message received by consumer
    
### 7. Is it possible to have a backoff strategy such as set a wait time for handling out-of-order delivery cases in Kafka (when idempotence is enabled), while ensuring that the broker does not get stuck waiting indefinitely?

**Instructor's**
- Yes - Retry & time backoff strategy
- Given the case of “1st (ok), 2nd (skipped), 3rd (new)”
    - 3rd message will be accepted, and 2nd message will be skipped with another sequence number
    - So will not be stuck indefinitely
--- 
 
Pending questions via email:
### 8. To further elaborate on the question I posted in class - "Is it possible to track specific skipped offsets in Kafka, particularly those missed due to log compaction?"
- The goal is to identify the specific list of offsets that a consumer has missed so that corrective actions can be taken, such as re-sending messages from the producer or flagging missing data.
- You mentioned in class about the auto.offset.reset flag. If set to none, the consumer can use a try-catch block to detect missing offsets, but this approach does not support auto-recovery or provide visibility into which offsets were skipped.
- Is there a way to determine exactly which offsets were lost and facilitate corrective measures?

### 9. Besides using Kafka Connect to implement a Dead Letter Queue (DLQ), what are other effective approaches for handling failed messages (both producer & consumer)?

### 10. How can a Kafka consumer retrieve the schemaID from a message?
- From my research, it seems that when a message is serialized using Confluent’s Avro Serdes, the first 5 bytes contain:
   - 1-byte magic byte (indicating schema usage)
   - 4-byte schema ID (corresponding to the registered schema in Schema Registry)
- Is this the correct and only way to extract the schema ID, or are there alternative approaches?
   - Would it be recommended to store the schema ID in the message headers as well?
- Additionally, is it possible for Kafka Control Center to extract and display the schema information for individual messages?

### 11. How should consumers handle schema changes in Schema Registry when transitioning from v1 to v2, especially if breaking changes are introduced? Understand there are the different schema compatibility modes that can be set, but in this case  I do not want to use defaultValue because there may not be a sensible default and I want to maintain explicit differences between v1 and v2.
- In this scenario, would it be a sensible approach for the consumer to read the schemaID from the message and dynamically use either v1 or v2 Avro-generated classes for deserialization?

### 12. What are the best integrations for batch processing with Kafka, and how do they compare in different use cases?



