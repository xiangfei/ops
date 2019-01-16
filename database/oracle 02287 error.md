Restrictions on Sequence Values You cannot use CURRVAL and NEXTVAL in the following constructs:
■ A subquery in a DELETE, SELECT, or UPDATE statement 
■ A query of a view or of a materialized view 
■ A SELECT statement with the DISTINCT operator 
■ A SELECT statement with a GROUP BY clause or ORDER BY clause    
■ A SELECT statement that is combined with another SELECT statement with the UNION, INTERSECT, or MINUS set operator 
■ The WHERE clause of a SELECT statement 
■ The DEFAULT value of a column in a CREATE TABLE or ALTER TABLE statement 
■ The condition of a CHECK constrain
resolve   insert into * from select   select