---
layout: post
title: First post
---

Next you can update your site name, avatar and other options using the _config.yml file in the root of your repository (shown below).

![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.

### Spark Data Source original API ###

https://developer.ibm.com/code/2016/11/10/exploring-apache-spark-datasource-api/

### Spark 2.3.1 Data Source V2 API Changes ###


* issues: https://issues.apache.org/jira/browse/SPARK-15689
* docs: https://docs.google.com/document/d/1n_vUVbF4KD3gxTmkNEon5qdQ-Z8qU5Frf6WMQZ6jJVM/edit#heading=h.mi1fbff5f8f9
* overall introduction: http://blog.madhukaraphatak.com/spark-datasource-v2-part-1/

* no data source interface (scala version) has been changed (sql/datasources/interfaces.scala
```scala
/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.apache.spark.sql.sources

import org.apache.spark.annotation.{DeveloperApi, Experimental, InterfaceStability}
import org.apache.spark.rdd.RDD
import org.apache.spark.sql._
import org.apache.spark.sql.catalyst.InternalRow
import org.apache.spark.sql.catalyst.expressions._
import org.apache.spark.sql.execution.streaming.{Sink, Source}
import org.apache.spark.sql.streaming.OutputMode
import org.apache.spark.sql.types.StructType

/**
 * Data sources should implement this trait so that they can register an alias to their data source.
 * This allows users to give the data source alias as the format type over the fully qualified
 * class name.
 *
 * A new instance of this class will be instantiated each time a DDL call is made.
 *
 * @since 1.5.0
 */
@InterfaceStability.Stable
trait DataSourceRegister {

  /**
   * The string that represents the format that this data source provider uses. This is
   * overridden by children to provide a nice alias for the data source. For example:
   *
   * {{{
   *   override def shortName(): String = "parquet"
   * }}}
   *
   * @since 1.5.0
   */
  def shortName(): String
}

/**
 * Implemented by objects that produce relations for a specific kind of data source.  When
 * Spark SQL is given a DDL operation with a USING clause specified (to specify the implemented
 * RelationProvider), this interface is used to pass in the parameters specified by a user.
 *
 * Users may specify the fully qualified class name of a given data source.  When that class is
 * not found Spark SQL will append the class name `DefaultSource` to the path, allowing for
 * less verbose invocation.  For example, 'org.apache.spark.sql.json' would resolve to the
 * data source 'org.apache.spark.sql.json.DefaultSource'
 *
 * A new instance of this class will be instantiated each time a DDL call is made.
 *
 * @since 1.3.0
 */
@InterfaceStability.Stable
trait RelationProvider {
  /**
   * Returns a new base relation with the given parameters.
   *
   * @note The parameters' keywords are case insensitive and this insensitivity is enforced
   * by the Map that is passed to the function.
   */
  def createRelation(sqlContext: SQLContext, parameters: Map[String, String]): BaseRelation
}

/**
 * Implemented by objects that produce relations for a specific kind of data source
 * with a given schema.  When Spark SQL is given a DDL operation with a USING clause specified (
 * to specify the implemented SchemaRelationProvider) and a user defined schema, this interface
 * is used to pass in the parameters specified by a user.
 *
 * Users may specify the fully qualified class name of a given data source.  When that class is
 * not found Spark SQL will append the class name `DefaultSource` to the path, allowing for
 * less verbose invocation.  For example, 'org.apache.spark.sql.json' would resolve to the
 * data source 'org.apache.spark.sql.json.DefaultSource'
 *
 * A new instance of this class will be instantiated each time a DDL call is made.
 *
 * The difference between a [[RelationProvider]] and a [[SchemaRelationProvider]] is that
 * users need to provide a schema when using a [[SchemaRelationProvider]].
 * A relation provider can inherit both [[RelationProvider]] and [[SchemaRelationProvider]]
 * if it can support both schema inference and user-specified schemas.
 *
 * @since 1.3.0
 */
@InterfaceStability.Stable
trait SchemaRelationProvider {
  /**
   * Returns a new base relation with the given parameters and user defined schema.
   *
   * @note The parameters' keywords are case insensitive and this insensitivity is enforced
   * by the Map that is passed to the function.
   */
  def createRelation(
      sqlContext: SQLContext,
      parameters: Map[String, String],
      schema: StructType): BaseRelation
}

/**
 * ::Experimental::
 * Implemented by objects that can produce a streaming `Source` for a specific format or system.
 *
 * @since 2.0.0
 */
@Experimental
@InterfaceStability.Unstable
trait StreamSourceProvider {

  /**
   * Returns the name and schema of the source that can be used to continually read data.
   * @since 2.0.0
   */
  def sourceSchema(
      sqlContext: SQLContext,
      schema: Option[StructType],
      providerName: String,
      parameters: Map[String, String]): (String, StructType)

  /**
   * @since 2.0.0
   */
  def createSource(
      sqlContext: SQLContext,
      metadataPath: String,
      schema: Option[StructType],
      providerName: String,
      parameters: Map[String, String]): Source
}

/**
 * ::Experimental::
 * Implemented by objects that can produce a streaming `Sink` for a specific format or system.
 *
 * @since 2.0.0
 */
@Experimental
@InterfaceStability.Unstable
trait StreamSinkProvider {
  def createSink(
      sqlContext: SQLContext,
      parameters: Map[String, String],
      partitionColumns: Seq[String],
      outputMode: OutputMode): Sink
}

/**
 * @since 1.3.0
 */
@InterfaceStability.Stable
trait CreatableRelationProvider {
  /**
   * Saves a DataFrame to a destination (using data source-specific parameters)
   *
   * @param sqlContext SQLContext
   * @param mode specifies what happens when the destination already exists
   * @param parameters data source-specific parameters
   * @param data DataFrame to save (i.e. the rows after executing the query)
   * @return Relation with a known schema
   *
   * @since 1.3.0
   */
  def createRelation(
      sqlContext: SQLContext,
      mode: SaveMode,
      parameters: Map[String, String],
      data: DataFrame): BaseRelation
}

/**
 * Represents a collection of tuples with a known schema. Classes that extend BaseRelation must
 * be able to produce the schema of their data in the form of a `StructType`. Concrete
 * implementation should inherit from one of the descendant `Scan` classes, which define various
 * abstract methods for execution.
 *
 * BaseRelations must also define an equality function that only returns true when the two
 * instances will return the same data. This equality function is used when determining when
 * it is safe to substitute cached results for a given relation.
 *
 * @since 1.3.0
 */
@InterfaceStability.Stable
abstract class BaseRelation {
  def sqlContext: SQLContext
  def schema: StructType

  /**
   * Returns an estimated size of this relation in bytes. This information is used by the planner
   * to decide when it is safe to broadcast a relation and can be overridden by sources that
   * know the size ahead of time. By default, the system will assume that tables are too
   * large to broadcast. This method will be called multiple times during query planning
   * and thus should not perform expensive operations for each invocation.
   *
   * @note It is always better to overestimate size than underestimate, because underestimation
   * could lead to execution plans that are suboptimal (i.e. broadcasting a very large table).
   *
   * @since 1.3.0
   */
  def sizeInBytes: Long = sqlContext.conf.defaultSizeInBytes

  /**
   * Whether does it need to convert the objects in Row to internal representation, for example:
   *  java.lang.String to UTF8String
   *  java.lang.Decimal to Decimal
   *
   * If `needConversion` is `false`, buildScan() should return an `RDD` of `InternalRow`
   *
   * @note The internal representation is not stable across releases and thus data sources outside
   * of Spark SQL should leave this as true.
   *
   * @since 1.4.0
   */
  def needConversion: Boolean = true

  /**
   * Returns the list of [[Filter]]s that this datasource may not be able to handle.
   * These returned [[Filter]]s will be evaluated by Spark SQL after data is output by a scan.
   * By default, this function will return all filters, as it is always safe to
   * double evaluate a [[Filter]]. However, specific implementations can override this function to
   * avoid double filtering when they are capable of processing a filter internally.
   *
   * @since 1.6.0
   */
  def unhandledFilters(filters: Array[Filter]): Array[Filter] = filters
}

/**
 * A BaseRelation that can produce all of its tuples as an RDD of Row objects.
 *
 * @since 1.3.0
 */
@InterfaceStability.Stable
trait TableScan {
  def buildScan(): RDD[Row]
}

/**
 * A BaseRelation that can eliminate unneeded columns before producing an RDD
 * containing all of its tuples as Row objects.
 *
 * @since 1.3.0
 */
@InterfaceStability.Stable
trait PrunedScan {
  def buildScan(requiredColumns: Array[String]): RDD[Row]
}

/**
 * A BaseRelation that can eliminate unneeded columns and filter using selected
 * predicates before producing an RDD containing all matching tuples as Row objects.
 *
 * The actual filter should be the conjunction of all `filters`,
 * i.e. they should be "and" together.
 *
 * The pushed down filters are currently purely an optimization as they will all be evaluated
 * again.  This means it is safe to use them with methods that produce false positives such
 * as filtering partitions based on a bloom filter.
 *
 * @since 1.3.0
 */
@InterfaceStability.Stable
trait PrunedFilteredScan {
  def buildScan(requiredColumns: Array[String], filters: Array[Filter]): RDD[Row]
}

/**
 * A BaseRelation that can be used to insert data into it through the insert method.
 * If overwrite in insert method is true, the old data in the relation should be overwritten with
 * the new data. If overwrite in insert method is false, the new data should be appended.
 *
 * InsertableRelation has the following three assumptions.
 * 1. It assumes that the data (Rows in the DataFrame) provided to the insert method
 * exactly matches the ordinal of fields in the schema of the BaseRelation.
 * 2. It assumes that the schema of this relation will not be changed.
 * Even if the insert method updates the schema (e.g. a relation of JSON or Parquet data may have a
 * schema update after an insert operation), the new schema will not be used.
 * 3. It assumes that fields of the data provided in the insert method are nullable.
 * If a data source needs to check the actual nullability of a field, it needs to do it in the
 * insert method.
 *
 * @since 1.3.0
 */
@InterfaceStability.Stable
trait InsertableRelation {
  def insert(data: DataFrame, overwrite: Boolean): Unit
}

/**
 * ::Experimental::
 * An interface for experimenting with a more direct connection to the query planner.  Compared to
 * [[PrunedFilteredScan]], this operator receives the raw expressions from the
 * `org.apache.spark.sql.catalyst.plans.logical.LogicalPlan`.  Unlike the other APIs this
 * interface is NOT designed to be binary compatible across releases and thus should only be used
 * for experimentation.
 *
 * @since 1.3.0
 */
@Experimental
@InterfaceStability.Unstable
trait CatalystScan {
  def buildScan(requiredColumns: Seq[Attribute], filters: Seq[Expression]): RDD[Row]
}
```

* Catalyst Scan
DataSourceStrategy choose what interface will be executed (prunedScan/FilteredPrunedScan/CatalystScan)
```scala
/**
 * A Strategy for planning scans over data sources defined using the sources API.
 */
case class DataSourceStrategy(conf: SQLConf) extends Strategy with Logging with CastSupport {
  import DataSourceStrategy._

  def apply(plan: LogicalPlan): Seq[execution.SparkPlan] = plan match {
    case PhysicalOperation(projects, filters, l @ LogicalRelation(t: CatalystScan, _, _, _)) =>
      pruneFilterProjectRaw(
        l,
        projects,
        filters,
        (requestedColumns, allPredicates, _) =>
          toCatalystRDD(l, requestedColumns, t.buildScan(requestedColumns, allPredicates))) :: Nil

    case PhysicalOperation(projects, filters,
                           l @ LogicalRelation(t: PrunedFilteredScan, _, _, _)) =>
      pruneFilterProject(
        l,
        projects,
        filters,
        (a, f) => toCatalystRDD(l, a, t.buildScan(a.map(_.name).toArray, f))) :: Nil

    case PhysicalOperation(projects, filters, l @ LogicalRelation(t: PrunedScan, _, _, _)) =>
      pruneFilterProject(
        l,
        projects,
        filters,
        (a, _) => toCatalystRDD(l, a, t.buildScan(a.map(_.name).toArray))) :: Nil

    case l @ LogicalRelation(baseRelation: TableScan, _, _, _) =>
      RowDataSourceScanExec(
        l.output,
        l.output.indices,
        Set.empty,
        Set.empty,
        toCatalystRDD(l, baseRelation.buildScan()),
        baseRelation,
        None) :: Nil

    case _ => Nil
  }

```

Only one of CatalystScan or PrunedFilteredScan should be chosen

```scala
// Based on Catalyst expressions. The `scanBuilder` function accepts three arguments:
  //
  //  1. A `Seq[Attribute]`, containing all required column attributes. Used to handle relation
  //     traits that support column pruning (e.g. `PrunedScan` and `PrunedFilteredScan`).
  //
  //  2. A `Seq[Expression]`, containing all gathered Catalyst filter expressions, only used for
  //     `CatalystScan`.
  //
  //  3. A `Seq[Filter]`, containing all data source `Filter`s that are converted from (possibly a
  //     subset of) Catalyst filter expressions and can be handled by `relation`.  Used to handle
  //     relation traits (`CatalystScan` excluded) that support filter push-down (e.g.
  //     `PrunedFilteredScan` and `HadoopFsRelation`).
  //
  // Note that 2 and 3 shouldn't be used together.
  private def pruneFilterProjectRaw(
    relation: LogicalRelation,
    projects: Seq[NamedExpression],
    filterPredicates: Seq[Expression],
    scanBuilder: (Seq[Attribute], Seq[Expression], Seq[Filter]) => RDD[InternalRow]): SparkPlan = {

    val projectSet = AttributeSet(projects.flatMap(_.references))
    val filterSet = AttributeSet(filterPredicates.flatMap(_.references))

    val candidatePredicates = filterPredicates.map { _ transform {
      case a: AttributeReference => relation.attributeMap(a) // Match original case of attributes.
    }}

    val (unhandledPredicates, pushedFilters, handledFilters) =
      selectFilters(relation.relation, candidatePredicates)

    // Combines all Catalyst filter `Expression`s that are either not convertible to data source
    // `Filter`s or cannot be handled by `relation`.
    val filterCondition = unhandledPredicates.reduceLeftOption(expressions.And)

    if (projects.map(_.toAttribute) == projects &&
        projectSet.size == projects.size &&
        filterSet.subsetOf(projectSet)) {
      // When it is possible to just use column pruning to get the right projection and
      // when the columns of this projection are enough to evaluate all filter conditions,
      // just do a scan followed by a filter, with no extra project.
      val requestedColumns = projects
        // Safe due to if above.
        .asInstanceOf[Seq[Attribute]]
        // Match original case of attributes.
        .map(relation.attributeMap)

      val scan = RowDataSourceScanExec(
        relation.output,
        requestedColumns.map(relation.output.indexOf),
        pushedFilters.toSet,
        handledFilters,
        scanBuilder(requestedColumns, candidatePredicates, pushedFilters),
        relation.relation,
        relation.catalogTable.map(_.identifier))
      filterCondition.map(execution.FilterExec(_, scan)).getOrElse(scan)
    } else {
      // A set of column attributes that are only referenced by pushed down filters.  We can
      // eliminate them from requested columns.
      val handledSet = {
        val handledPredicates = filterPredicates.filterNot(unhandledPredicates.contains)
        val unhandledSet = AttributeSet(unhandledPredicates.flatMap(_.references))
        AttributeSet(handledPredicates.flatMap(_.references)) --
          (projectSet ++ unhandledSet).map(relation.attributeMap)
      }
      // Don't request columns that are only referenced by pushed filters.
      val requestedColumns =
        (projectSet ++ filterSet -- handledSet).map(relation.attributeMap).toSeq

      val scan = RowDataSourceScanExec(
        relation.output,
        requestedColumns.map(relation.output.indexOf),
        pushedFilters.toSet,
        handledFilters,
        scanBuilder(requestedColumns, candidatePredicates, pushedFilters),
        relation.relation,
        relation.catalogTable.map(_.identifier))
      execution.ProjectExec(
        projects, filterCondition.map(execution.FilterExec(_, scan)).getOrElse(scan))
    }
  }
```
