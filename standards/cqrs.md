# CQRS Pattern
See http://martinfowler.com/bliki/CQRS.html

This pattern is used by the dataset service. CRUD operations are completed against a relational store, and query operations are completed against a de-normalized representation stored in SOLR.