# Maltego — Link Analysis and OSINT Graphs

## 1. Purpose
Maltego visualizes relationships between entities such as domains, DNS names, IP addresses, organizations, and email addresses using transforms/data sources.

**Memory hook:** Maltego = **a detective's connection board**.

## 2. Inputs
- Starting entity, such as a domain you are authorized to investigate.
- Selected transform (a query that retrieves related data).
- Data-source access, account, or API configuration as required by the edition.

## 3. GUI workflow
1. Open Maltego and create a graph.
2. Add a starting entity, e.g. a domain.
3. Select it and run a relevant transform.
4. Inspect resulting nodes and connecting edges.
5. Open entity details and source/provenance information.
6. Save the graph and document which transforms generated the results.

## 4. Output and interpretation
- **Node:** an entity such as a domain, IP, or organization.
- **Edge/link:** a relationship asserted by the transform/data source.
- **Transform:** the operation used to discover related information.
- **Graph:** visual representation of collected relationships.

Explain: “The graph shows relationships returned by the selected data source and transform. A link is a lead to investigate, not automatic proof of common ownership or wrongdoing.”

## 5. Common issues
- No transform results: provider may need a login/API key, be unavailable, or have no data.
- Many nodes: filter and group the graph rather than treating every node as equally important.
- Conflicting records: preserve source details and validate independently.

## 6. Viva
**Q: What is OSINT?** Intelligence gathered from publicly available sources.
**Q: What is a transform?** A query/action that retrieves related entities.
**Q: What does an edge mean?** A relationship reported by the data source; strength depends on its semantics and reliability.
**Q: Why visualize data?** It helps reveal patterns and relationships that are harder to see in a flat list.
