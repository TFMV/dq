# Designed, Then Written

## What DQOps Taught Me About Data Quality and Documentation

> "Our documentation was 'not written.' It was first 'designed,' and we filled in the content."
> — Piotr Czarnas, creator of DQOps

In most engineering teams, documentation is an afterthought. It's what you slap together before launch—or worse, after. A formality. A chore.

But DQOps, a modern data quality platform, flips that script. Its documentation doesn't feel like an appendix—it feels like part of the product. Structured, clear, and deeply intentional. It reads like something built with the same precision as the system it describes.

And that made something click for me.

Maybe the reason DQOps gets data quality right is the same reason its documentation stands out: both were designed—not just assembled.

The best documentation, like the best data pipelines, is built on trust, transparency, and structure. And DQOps doesn't just document those values—it embodies them.

## Docs That Don't Just Describe—They Deliver

Most open source projects evolve in bursts—features come fast, and documentation lags behind, gasping for air. Developers learn to live with it. You skim the README, squint at a half-written guide, and hope the source code tells the rest of the story.

DQOps breaks that pattern.

Its documentation doesn't feel bolted on. It feels baked in. From the moment you open the repo, it's clear: this system was documented as it was built, not after the fact. The /docs directory isn't a dumping ground—it's a structured knowledge system. Architecture, workflows, configuration, rule design—it's all there, organized intentionally.

They even have a documentation guide that serves as a kind of table of contents for the reader's mental model. You don't have to guess where to start. You don't have to reverse-engineer the system just to get your bearings.

The architecture docs are a standout. Instead of listing components in isolation, they present the system as a living whole—complete with deployment topologies and real-world scenarios. The diagrams don't just show what is—they suggest what could be. That's rare.

This kind of documentation helps everyone. New users get an on-ramp. Power users get a framework to think through extensions and edge cases. And maintainers get fewer support questions because the answers are already where they belong.

DQOps treats documentation like an interface—one that's just as important as the CLI or the UI.

## Architecture That Embodies Design-First Principles

The architecture of DQOps reveals the same design-first principles that guide its documentation. Unlike monolithic data quality tools, DQOps is built with clear separation of concerns that makes its capabilities both powerful and approachable.

Consider how DQOps structures its core components:

1. **DQOps Runtime Engine**: A unified Java/Spring Boot engine that houses the web server, REST API, and command-line interface. This core engine orchestrates all data quality operations while presenting multiple interfaces for different user preferences.

2. **User Home Structure**: Rather than hiding configurations in an opaque database, DQOps stores everything in a transparent, version-controllable directory structure. YAML files define connections, tables, and checks, making them easily manageable through Git and CI/CD pipelines.

3. **Modular Data Storage**: Results are organized in Parquet files under a `.data` directory, creating a lightweight "data quality data lake." This architecture choice enables offline operation and easy inspection of historical metrics without requiring external databases.

The brilliance of this design isn't just technical—it's experiential. By structuring the system around clean boundaries and simple interfaces, DQOps makes complex data quality tasks accessible. For example, you can add a custom sensor (a SQL template for measuring something in your data) without needing to understand the entire codebase; you simply place a properly formatted YAML file in the `sensors/` directory.

This modular design mirrors how well-structured documentation makes complex topics digestible—by presenting clear boundaries, relationships, and entry points rather than an undifferentiated mass of information.

## The "Zero to Monitoring" Experience

Where DQOps truly shines is in transforming complex data quality processes into streamlined workflows. Setting up continuous monitoring—a task that might take weeks with other tools—becomes remarkably straightforward:

1. **Connection and Discovery**: After a simple installation (`pip install dqops`), connecting to a data source requires just basic credentials. DQOps then auto-discovers your tables and columns, creating a metadata model without manual configuration:

   ```
   python -m dqops
   # Then in the UI: Add Connection > Enter credentials > Import tables
   ```

2. **Automated Rule Mining**: Perhaps the most impressive capability is how DQOps automatically suggests appropriate quality checks based on your data profile. When you run initial profiling, the platform analyzes your data's patterns and distributions, then proposes checks with reasonable thresholds:

   ```
   # Automatically generates checks like:
   # - "daily row count shouldn't drop below historical average by more than 10%"
   # - "null percentage in email column shouldn't exceed 2%"
   # - "distinct count of category column should remain between 5-15"
   ```

   This eliminates the traditional guesswork of setting thresholds and drastically reduces setup time.

3. **Schedule Once, Monitor Continuously**: With checks configured, enabling continuous monitoring requires just setting a schedule. DQOps handles the rest—executing checks, comparing results against rules, grouping similar issues into incidents, and sending notifications:

   ```
   # In connection settings:
   # Profiling checks: "0 12 1 * *" (monthly at noon on the 1st)
   # Monitoring checks: "0 12 * * *" (daily at noon)
   ```

The elegance of this workflow stems directly from DQOps's designed-first approach. The architecture enables a journey from connection to continuous monitoring that feels natural and progressive. Each step builds logically on the previous one, with the platform handling complexity behind clean interfaces.

Integration with existing pipelines is equally streamlined. DQOps provides Airflow operators that can be dropped into DAGs to validate data at critical points:

```python
# In an Airflow DAG:
from dqops.airflow import RunChecksOperator

validate_data = RunChecksOperator(
    task_id="validate_sales_data",
    connection_name="sales_dw",
    table_name="fact_sales",
    check_categories=["monitoring"],
    dag=dag
)

load_task >> validate_data >> downstream_task
```

This simplicity belies the sophisticated engine working behind the scenes—again reflecting how well-designed documentation presents complex concepts in manageable pieces.

## Where Documentation Meets Implementation

To truly appreciate how DQOps's design-first approach shines through both documentation and functionality, let's examine one specific feature: the "number found in set percent" data quality check.

Consider a common data challenge: ensuring values in a numeric column conform to an expected set—perhaps status codes, category identifiers, or rating scales. In traditional tools, this would require custom SQL queries for measurement, separate logic for validation, and yet more code for alerting.

DQOps's documentation for this check illustrates their unified approach perfectly:

1. **Clear, Contextual Definition**: The documentation begins with a precise explanation: "A column-level check that calculates the percentage of rows for which the tested numeric column contains a value from a set of expected values." It immediately adds business context: "This check is useful for columns that store numeric codes (such as status codes)."

2. **Visual Organization**: The information is structured in a logical hierarchy—from overview to variants (profiling, daily, monthly) to implementation details. Navigation is intuitive with a clear table of contents.

3. **Multi-Format Examples**: The documentation provides examples in multiple forms: YAML configuration, command-line commands, and most impressively, the actual SQL templates for different database systems (MySQL, SQL Server, Teradata, Trino).

What's remarkable is how the documentation matches the system's implementation:

```yaml
# From the documentation - YAML configuration example
columns:
  target_column:
    profiling_checks:
      accepted_values:
        profile_number_found_in_set_percent:
          parameters:
            expected_values: [2, 3]
          warning:
            min_percent: 95
          error:
            min_percent: 90
```

This YAML isn't just an example—it's the actual configuration you'd use. Notice how readable it is—a clear hierarchy that mirrors the mental model: column > check type > specific check > parameters > threshold levels.

Behind this simple configuration is a sophisticated implementation. The documentation reveals the actual SQL template:

```sql
SELECT
  CASE
    WHEN COUNT(analyzed_table.`target_column`) = 0 THEN 100.0
    ELSE 100.0 * SUM(
      CASE
        WHEN analyzed_table.`target_column` IN (2, 3)
          THEN 1
        ELSE 0
      END
    ) / COUNT(analyzed_table.`target_column`)
  END AS actual_value,
  ...
```

This transparency is unprecedented. Most platforms hide their implementation details, but DQOps exposes them—not just as reference material but as learning tools. You can understand not just what the check does but how it works.

The brilliance of this approach is that despite exposing complexity, using the feature remains simple. A data engineer can configure this check in seconds through the UI or a few lines of YAML, yet still have access to the complete technical details when needed.

This pattern—clear conceptual explanation, intuitive configuration, and transparent implementation—repeats across hundreds of checks in the documentation. It's a masterclass in how to document complex functionality in a way that serves both novice and expert users.

## The Architecture of Trust

The parallels between well-designed documentation and effective data quality practices became increasingly apparent as I worked with DQOps. Both share foundational principles:

1. **Structure creates clarity**: Just as DQOps documentation is organized with clear navigation and relationships between concepts, their data quality engine separates concerns into distinct layers—sensors (SQL templates that measure metrics), rules (evaluators that apply thresholds), and checks (unified configurations that combine both). This modularity makes complex quality requirements manageable.

2. **Intention precedes implementation**: DQOps documentation was clearly mapped out before being filled with content. Similarly, their approach to data quality emphasizes defining what "good data" looks like before writing a single validation. Their Rule Mining feature analyzes data to suggest appropriate validations—a design-first approach to data quality.

3. **Usability trumps comprehensiveness**: Rather than overwhelming users with every possible detail, DQOps documentation prioritizes practical workflows. Their data quality system applies the same principle—rather than scanning entire datasets with every possible check, they've designed incremental, optimized evaluations that can run efficiently in production.

The synergy between these approaches isn't coincidental. DQOps has fundamentally reimagined what data quality tools could be by applying the same design principles to both product and documentation.

## Designed to Teach, Not Just Explain

Data quality is messy. It straddles business logic, technical validation, and statistical nuance. And learning a new platform in this space? Even messier. It's easy to get lost before you've even found your footing.

What makes DQOps stand out is how its documentation meets you where you are—and guides you forward.

Take the architecture docs. They don't start with a flood of implementation details. They start with a bird's-eye view—how the pieces fit, how they talk to each other, how the whole thing works. Only after you've built that mental map do the specifics come into focus.

The same rhythm shows up across the board. Checks, sensors, rules—each introduced with clarity and context before diving into mechanics. You're never just handed a config file and told "figure it out." There's a deliberate arc: understand, define, validate. It mirrors the very way DQOps approaches data quality itself.

Even the way you move through the docs feels intentional. Related ideas are linked. Examples are practical, not toy problems. References and walkthroughs are clearly separated so you're not flipping between use case and spec.

This isn't just good documentation. It's good pedagogy. It's a system designed to teach, not just inform.

## Enterprise Clarity in a World of Data Complexity

For enterprise users accustomed to brittle data tooling with impenetrable documentation, DQOps offers a refreshing alternative. Complex systems don't have to be complicated to use when documentation is designed with user needs in mind.

This philosophy extends to how DQOps approaches enterprise data quality challenges. While many platforms overwhelm users with technical minutiae, DQOps provides clear pathways for common workflows:

- Connecting to data sources and importing metadata
- Profiling data to establish a baseline understanding
- Configuring quality checks with appropriate thresholds
- Scheduling regular validation and alerting
- Reporting on quality metrics through dashboards

Each step is documented with the same clarity and intention as the product itself. The result is a platform that doesn't just validate data—it makes data quality legible to the entire organization.

This legibility is perhaps DQOps's most significant contribution. Data quality isn't just a technical concern; it's a business imperative that spans roles and departments. By designing documentation that speaks to different stakeholders—from data engineers to business analysts to executives—DQOps bridges the gap between technical implementation and business value.

## When Documentation Is the Product

The ultimate insight from DQOps is that documentation isn't separate from the product—it is the product. Or at least, it's an essential facet of the product experience that deserves the same design thinking and refinement as any feature or interface.

This perspective explains why their documentation feels so different from typical open source projects. It wasn't written to satisfy a requirement or check a box. It was designed as a first-class citizen, with user journeys and information architecture carefully considered before implementation.

For data teams building their own tools or processes, this paradigm shift offers a valuable lesson. Treating documentation as design rather than implementation changes how we approach knowledge sharing. Instead of asking "What do we need to document?" we should ask "How will users experience this knowledge?"

The same applies to data quality initiatives. Rather than jumping straight to implementing checks and validations, we should first design the quality framework—defining principles, metrics, and workflows that make sense for our specific context.

## Designed, Then Built

What DQOps gets right isn't just about documentation. It's a broader truth that echoes across software, data, and the systems that bind them: design-first thinking leads to better outcomes.

Start with structure. Prioritize clarity, intent, and usability. That's how you build systems that hold up under pressure. Systems that are easier to understand, easier to extend, and far less likely to surprise us in production.

That applies to documentation. It applies to data quality. It applies to almost everything we ship.

The next time you're tempted to bolt on docs after the fact, or slap in a data check without thinking through the bigger picture, remember what DQOps quietly demonstrates:

The strongest systems aren't patched together.
They're designed first and written second.
