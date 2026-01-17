# iTOP Platform - Service Testing Guide

## Prerequisites
Ensure all services are running:
```bash
docker-compose up -d
docker-compose ps
```

Expected output: All containers should show `Up` status.

## Service Health Checks

### PostgreSQL (Port 5432)
```bash
docker exec itop-postgres psql -U admin -d itop -c "SELECT version();"
```
Expected: PostgreSQL version info displayed.

### MongoDB (Port 27017)
```bash
docker exec itop-mongodb mongo --username admin --password admin123 --authenticationDatabase admin --eval "db.adminCommand('ping')"
```
Expected: `{ "ok" : 1 }`

### Redis (Port 6379)
```bash
docker exec itop-redis redis-cli ping
```
Expected: `PONG`

### Zookeeper (Port 2181)
```bash
docker exec itop-zookeeper echo ruok | nc localhost 2181
```
Expected: `imok`

### Kafka (Port 9092)
```bash
# List topics
docker exec itop-kafka kafka-topics --bootstrap-server kafka:9092 --list

# Create test topic
docker exec itop-kafka kafka-topics --bootstrap-server kafka:9092 --create --topic test-topic --partitions 1 --replication-factor 1

# Produce message
echo "test message" | docker exec -i itop-kafka kafka-console-producer --broker-list kafka:9092 --topic test-topic

# Consume message
docker exec itop-kafka kafka-console-consumer --bootstrap-server kafka:9092 --topic test-topic --from-beginning --max-messages 1
```
Expected: Test message appears in consumer output.

## Network Connectivity Test
Verify services can communicate via `itop-network`:
```bash
# From Kafka container, test PostgreSQL connectivity
docker exec itop-kafka nc -zv postgres 5432

# From Kafka container, test MongoDB connectivity
docker exec itop-kafka nc -zv mongodb 27017
```
Expected: Connection successful messages.

## Quick Validation Script
```bash
#!/bin/bash
echo "=== iTOP Platform Service Health Check ==="
docker exec itop-postgres psql -U admin -d itop -c "SELECT 'PostgreSQL: OK';" 2>/dev/null && echo "✓ PostgreSQL" || echo "✗ PostgreSQL"
docker exec itop-mongodb mongo --username admin --password admin123 --authenticationDatabase admin --eval "db.adminCommand('ping')" 2>/dev/null | grep -q "ok" && echo "✓ MongoDB" || echo "✗ MongoDB"
docker exec itop-redis redis-cli ping 2>/dev/null | grep -q "PONG" && echo "✓ Redis" || echo "✗ Redis"
docker exec itop-zookeeper echo ruok | nc localhost 2181 2>/dev/null | grep -q "imok" && echo "✓ Zookeeper" || echo "✗ Zookeeper"
docker exec itop-kafka kafka-topics --bootstrap-server kafka:9092 --list 2>/dev/null > /dev/null && echo "✓ Kafka" || echo "✗ Kafka"
echo "=== All checks complete ==="
```

Save as `test-services.sh` and run: `bash test-services.sh`
