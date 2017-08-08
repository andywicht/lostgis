# LostGIS

## Types

### TPV

TPV is Time, Position, Velocity storage type.

    create type TPV as (
        -- position
        geom     geometry(point, 3857),
        accuracy float,
        -- velocity
        heading  float,
        speed    float,
        -- time
        ts       timestamptz,
        -- helpers
        source   text,
        osm_id   bigint
    );


### opening_hours

Implementation for OpenStreetMap opening_hours type. Allows checking whether it's open or closed now, for simple cases.

    create type opening_hours as (
        human_readable text,
        is_24          boolean,
        is_valid       boolean,
        week_mask      bit(10080)
    );

## Functions

### overlaps(timestamp, opening_hours)

Check whether timestamp `opening_hours` are open at `timestamp`.

 * note `timestamp`, not `timestamptz` - you should convert timezone yourself, possibly with a geographical look up;
 * `opening_hours` need to be casted via `text` as you cannot create custom input function for type in Postgres;
 * only simple cases (weekdays and hours-minutes) are currently supported.


    select overlaps(
       '2017-08-13 13:00':: timestamp,
       'Mo-Fr 05:00-15:00,19:00-21:00; Sa 05:00-12:00,14:00-21:00; Su 05:00-14:00,17:00-21:00' :: text :: opening_hours
    )


### coslat

Get latitude cosine. Works on projected geometries too.

    function coslat(geometry) returns float

### coslat (tpv)

Get cosine from latitude.

    function coslat(tpv) returns float

Direct calculation of `cos(lat)` in `3857` without reprojecting it to `4326`. Used following expression `coslat = cos(asin(tanh(Y / 6378137)))`.

### median

Aggregate function for getting of median

    aggregate median( anyelement )

### project_position

Get new position for given `tpv` and new `timestamp`.

    function project_position(TPV, timestamptz) returns TPV

Calculate new position based on `speed` and `heading` from given `tpv`.


### ST_AddTime

Add time to array of TPV based on interpolation between start and stop time.

    create or replace function ST_AddTime(
        tpvarray             tpv [],
        start_time           timestamptz,
        end_time             timestamptz,
        interpolation_method text default 'length' -- 'length', 'count'
    ) returns tpv []


### ST_AnglesEqual

Comparison of two angles

    function ST_AnglesEqual(
        angle1 float,
        angle2 float,
        oneway boolean,
        delta  float default pi() / 4
    ) returns boolean

### ST_AnglesEqualD

Comparison of two angles in degrees.

    function ST_AnglesEqualD(
        angle1 float,
        angle2 float,
        oneway boolean,
        delta  float default 45
    ) returns boolean

### ST_Fast_Real_Buffer

It gets buffer in real meters, in contrast to [ST_Buffer](http://www.postgis.org/docs/ST_Buffer.html) operating in projection units.

    function ST_Fast_Real_Buffer(
        geom geometry, radius float,
        buffer_style_parameters text default ''
    ) returns geometry

### ST_Fast_Real_Length

It gets length in real meters, in contrast to [ST_Length](http://www.postgis.org/docs/ST_Length.html) operating in projection units.

    function ST_Fast_Real_Length(
        geom geometry
    ) returns double precision

### ST_FilterSmallRings

Leaves only large rings in polygon geometry. Useful for map generalization.

    function ST_FilterSmallRings(
        geom     geometry,
        min_area float default 0
    ) returns geometry

### ST_GridCell

Get the geometry of rectangular cell of grid.

    function ST_GridCell(
        point geometry,
        grid_size float default 500
    ) returns geometry

Useful for binning point datasets.

### ST_LargestSubPolygon

Leave a single polygon from a multipolygon geometry.

    function ST_LargestSubPolygon(
        geom geometry
    ) returns geometry

Useful for map generalization.

### ST_LineAngleAtPoint

Given a line and a point at it, find the azimuth of segment the point is closest to.

    function ST_LineAngleAtPoint(
        point geometry,
        line  geometry,
        delta float default 1
    ) returns float

### ST_RealOffsetCurve

Return an offset line at a given distance and side from an input line. Radius is in signed value meters.

    function ST_RealOffsetCurve(
        geom geometry,
        radius float,
        buffer_style_parameters text default ''
    ) returns geometry

### ST_Safe_Difference

Replacement for [ST_Difference](http://www.postgis.org/docs/ST_Difference.html), automatically repairing invalid geometries (see also `ST_Safe_Repair`). 

    function ST_Safe_Difference(
        geom_a           geometry,
        geom_b           geometry default null,
        message          text default '[unspecified]',
        grid_granularity double precision default 1
    ) returns geometry

Also `ST_Safe_Difference(geom, null) = geom`, which is useful in aggregations.

### ST_Safe_Intersection

Replacement for [ST_Intersection](http://www.postgis.org/docs/ST_Intersection.html), automatically repairing invalid geometries (see also `ST_Safe_Repair`).

    function ST_Safe_Intersection(
        geom_a           geometry,
        geom_b           geometry default null,
        message          text default '[unspecified]',
        grid_granularity double precision default 1
    ) returns geometry

### ST_Safe_Repair

Function that tries hard to get a valid geometry out of any geometry.

    function ST_Safe_Repair(
        geom    geometry,
        message text default '[unspecified]'
    ) returns geometry

### ST_TimeLineMerge

Sew together an array of segments of track, where (x,y,z) is mapped to (x,y,timestamp).

    function ST_TimeLineMerge(
        geoms geometry ( linestringz ) []
    ) returns setof geometry ( linestringz )
