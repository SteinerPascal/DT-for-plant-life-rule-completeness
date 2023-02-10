# Testing Rule soundness with SPARQL for simble Event-Condition-Action (ECA) rules

This repository demonstrates the possibility to implement the proposed algorith in the paper Conflict Detection in Rule Based IoT Systems (https://ieeexplore.ieee.org/document/8936266) with SPARQL.

This requires rules which are described in RDF and loaded in a triplestore. It is then possible to analyze a group of rules on different type of conflicts:

- Execution Conflict
- Shadow Conflict
- Independent Conflict

For more information about the differnt types of conflicts. Read the above mentioned paper.

## Requirments
- Triplestore which supports SPARQL >= v1.1

## Execute experiment
1. Load ontology file (./ontology/conflict-detection) into triplestore. This file contains the TBox and ABox information. 

2. Execute queries for conflict detections
Independent conflict
```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX fcd: <http://www.semanticweb.org/z0045r1r/ontologies/2023/fcd#>
select * where { 
	?rule1 rdf:type fcd:Rule.
    ?rule2 rdf:type fcd:Rule.
    FILTER(?rule1 != ?rule2)
    ?rule1 fcd:hasClause ?clause1.
    ?rule2 fcd:hasClause ?clause2.
    #Conflicting Action
    ?clause1 fcd:hasAction ?action1.
    ?clause2 fcd:hasAction ?action2.
    FILTER(?action1 != ?action2)
    #NOT Similar sensors
    ?clause1 fcd:hasSensor ?sens1.
    ?clause2 fcd:hasSensor ?sens2.
    FILTER(?sens1 != ?sens2)
    
} limit 100 
```

Shadow conflict 
```

PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX fcd: <http://www.semanticweb.org/z0045r1r/ontologies/2023/fcd#>
select * where { 
	?rule1 rdf:type fcd:Rule.
    ?rule2 rdf:type fcd:Rule.
    FILTER(?rule1 != ?rule2)
    ?rule1 fcd:hasClause ?clause1.
    ?rule2 fcd:hasClause ?clause2.
    #Conflicting Action
    ?clause1 fcd:hasAction ?action1.
    ?clause2 fcd:hasAction ?action2.
    FILTER(?action1 = ?action2)
    #Similar sensors
    ?clause1 fcd:hasSensor ?sens1.
    ?clause2 fcd:hasSensor ?sens2.
    #FILTER(?sens1 = ?sens2)
    #Sub range
    ?clause1 fcd:hasTriggerCondition ?cond1.
    ?clause2 fcd:hasTriggerCondition ?cond2.
    OPTIONAL{?cond1 fcd:hasLowerBoundary ?cond1LowerBound.}
    OPTIONAL{?cond2 fcd:hasLowerBoundary ?cond2LowerBound.}
    OPTIONAL{?cond1 fcd:hasUpperBoundary ?cond1UpperBound.}
    OPTIONAL{?cond2 fcd:hasUpperBoundary ?cond2UpperBound.}
    
    {
         FILTER(?cond1UpperBound >= ?cond2UpperBound)
    } UNION
    {
       FILTER(?cond1LowerBound <= ?cond2LowerBound) 
    }
    
    
} limit 100 
```

Execution conflict (overlapping range)
```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX fcd: <http://www.semanticweb.org/z0045r1r/ontologies/2023/fcd#>
select * where { 
	?rule1 rdf:type fcd:Rule.
    ?rule2 rdf:type fcd:Rule.
    FILTER(?rule1 != ?rule2)
    ?rule1 fcd:hasClause ?clause1.
    ?rule2 fcd:hasClause ?clause2.
    #Conflicting Action
    ?clause1 fcd:hasAction ?action1.
    ?clause2 fcd:hasAction ?action2.
    FILTER(?action1 != ?action2)
    #Similar sensors
    ?clause1 fcd:hasSensor ?sens1.
    ?clause2 fcd:hasSensor ?sens2.
    FILTER(?sens1 = ?sens2)
    #Overlapping Range
    ?clause1 fcd:hasTriggerCondition ?cond1.
    ?clause2 fcd:hasTriggerCondition ?cond2.
    ?cond1 fcd:hasUpperBoundary ?cond1UpperBound.
    ?cond2 fcd:hasLowerBoundary ?cond2LowerBound
    FILTER(?cond1UpperBound >= ?cond2LowerBound)
} limit 100 
```

# Results

The following conflicts should be recognized

|  Conflicting Rules | Conflict Type  |  Conflict Explanation |
|--------------------|----------------|-----------------------|
| 1 & 2   | Execution Conflict  | If the soil humidity is within 30-35, then two contradictive actuations are executed.  |   
| 1 & 2  | Shadow Conflict  | The anti-trigger condition of rule 2 is redundant to rule 1. |   
| 1 & 3  | Shadow Conflict  | The anti-trigger condition of rule 3 is redundant to rule 1  |   
| 2 & 3  | Shadow Conflict  | The anti-trigger condition of rule 3 is redundant to the anti-trigger condition of rule 2.  |   
| 1 & 3  | Independent Conflict  | If the soil humidity is above 30 and the temperature is above 33, then two conflicting actuations are executed  |   
| 2 & 3  | Independent Conflict  | If the soil humidity is above 50 and the temperature above 33 then two conflicting actuations are executed. 
  |   
