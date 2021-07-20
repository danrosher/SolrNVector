# Solr NVector

## Why N-Vector

[N- vector](https://en.wikipedia.org/wiki/N-vector) is the outward-pointing unit vector that is normal in that position
 to an ellipsoid.
 
 In particular it can be used to calculate the 
 [Great circle distance](https://en.wikipedia.org/wiki/N-vector#Example:_Great_circle_distance)
 Mathematically the simplest is arccos(Na . Nb), arc cosine of the dot product of two n-vectors.
 
 This implementation make use of  [FasterInvTrig](https://github.com/danrosher/FasterInvTrig)
 that uses an approximation of arccos that's suitable for extremely fast distance calculations.
 
 N-Vector then can be a much faster implemtation to determine the Great circle distance than say 
 Haversine which is  commonly used in Solr for example.
 
 ## Installation
 
 ./gradlew clean jar
 
 ### Add to solrconfig.xml
 
 e.g 
 
    <config>
    ... 
    <lib dir="${solr.install.dir:../../../..}/dist/" regex="SolrNVector-1.0-SNAPSHOT.jar" />
    ....
    <valueSourceParser name="nvdist" class="com.github.danrosher.solr.search.NvectorValueSourceParser" />
    
    # "nvdist" can be any string
   
 ### Add to schema.xml
 
 e.g.
 
    <fieldType name="nvector" class="com.github.danrosher.solr.schema.NVector" subFieldSuffix="_d"/>
    <field name="nv" type="nvector" indexed="true" stored="true" multiValued="false"/> 
    <dynamicField name="*_d" type="double" indexed="true" stored="false"/> 
    
 ## Usage
 
    #Here 52.01966071979866, -0.4983083573742952 is the query point
 
 ### Get distance in fl, distance in km (nv is the nvector field)
    q=*:*&fl=*,id,dist:nvdist(52.01966071979866, -0.4983083573742952,nv)        
    
 ### Filter by distance in km (here we set the upper limit to 30km)
    q={!frange u=30}nvdist(52.01966071979866, -0.4983083573742952,nv)
 
 ### Sort by distance
    q=*:*&sort=nvdist(52.01966071979866, -0.4983083573742952,nv) asc  #closest first
    q=*:*&sort=nvdist(52.01966071979866, -0.4983083573742952,nv) desc #furthest first
    
### Combine them all
    q={!frange u=30}nvdist(52.01966071979866, -0.4983083573742952,nv)
    &fl=*,id,dist:nvdist(52.01966071979866, -0.4983083573742952,nv) 
    &sort=nvdist(52.01966071979866, -0.4983083573742952,nv) asc  #closest first
    
### Combine them all with variable substitution  
    dist=nvdist(52.01966071979866, -0.4983083573742952,nv)  
    q={!frange u=30}$dist
    &fl=*,id,dist:$dist
    &sort=$dist asc  #closest first
    
OR
    
    lat=52.01966071979866
    &lon=-0.4983083573742952
    
    &dist=nvdist($lat,$lon,nv)  
    q={!frange u=30}$dist
    &fl=*,id,dist:$dist
    &sort=$dist asc  #closest first
        
    


    
    
      
 
 
 
 