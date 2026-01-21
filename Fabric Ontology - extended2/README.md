# Sample set for Ontology / Graph

## Sample queries

### A list of production line IDs 

```
MATCH (node_Line:ProductionLine)
RETURN node_Line.id
LIMIT 100
```

do you want bubbles?

```
MATCH (node_Line:ProductionLine)
RETURN TO_JSON_STRING(node_Line) AS `ProductionLine`
```

(do not forget to do the JSON thing on relationships too, otherwise you only get bubbles, no arrows)

### A single production line via filter

```
MATCH (node_Line:ProductionLine)
WHERE node_Line.grade = "A+"
RETURN node_Line.id
```

### Get combination of operator and productionline where grade equals 'A+' (not unique)

A relationship is introduced.

```
MATCH (node_Operator:Operator)-[edge_Operates:Operates]->(node_Line:ProductionLine)
WHERE node_Line.grade = "A+"
RETURN node_Operator.name, node_Line.id
```

Alternative, using a LET for parameters:

```
MATCH (node_Operator:Operator)-[edge_Operates:Operates]->(node_Line:ProductionLine)
LET grade = node_Line.grade
FILTER grade = "A+"
RETURN node_Operator.name, node_Line.id
```

Or, turn the arrow other way around:

```
MATCH (node_Line:ProductionLine)<-[edge_Operates:Operates]-(node_Operator:Operator)
WHERE node_Line.grade = "A+"
RETURN node_Operator.name, node_Line.id, edge_Operates.dayOfWeek
```

same result.

As an alternative filter, put in the node description:

```
MATCH (node_Line:ProductionLine where node_Line.grade = "A+")<-[edge_Operates:Operates]-(node_Operator:Operator)
RETURN node_Operator.name, node_Line.id, edge_Operates.dayOfWeek
```

### Get the relationship between Operators and Technicians

Without telling about the relationship:

```
MATCH TRAIL (node_Technician:Technician)-[]-{1, 3}(node_Operator:Operator where node_Operator.name = "Operator 1")
RETURN DISTINCT node_Operator.name, node_Technician.name
ORDER BY node_Operator.name, node_Technician.name
```

The actual relationship is: Operator -> Productionline -> Workorder -> Technician

Technician 1 only worked workorders for Line 1. Operator 1 only operated Line 1. 

Be careful with the min/max number of hops.

### Get managers self-relationship for operators

```
MATCH (node_Operator1:Operator)-[]->{1,1}(node_Operator2:Operator)
RETURN node_Operator1.id, "Manages" as role, node_Operator2.id
ORDER BY node_Operator1.id, node_Operator2.id
```

Notice the arrow? This is to provide direction.

without arrow, you get all the possible relationships, including two-way duplicates (here, using DISTINCT does not work).

```
MATCH (node_Operator1:Operator)-[]-{1,1}(node_Operator2:Operator)
RETURN node_Operator1.id, "Manages" as role, node_Operator2.id
ORDER BY node_Operator1.id, node_Operator2.id
```

