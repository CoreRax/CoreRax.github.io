下面是meta-data的基本结构：    

     <meta-data android:name="string"

           android:resource="resource specification"

           android:value="string" /> 
可以包含在<activity> <activity-alias> <service> <receiver>四个元素中。  
用name-value对的格式给其父组件提供任意可选的数据。一个组件元素能都包含任意数量的meta-data子元素。他们所有的值都会被收集在Bundle对象中并且使其可以作为组件的Packageiteminfo.metaData字段。通常值是通过其value属性来指定的。但是，也可以使用resource属性来代替，把一个资源ID跟值进行关联。
例如，下面的代码就是把存储在@string/kangaroo资源中的值跟”zoo”名称进行关联：

    <meta-data android:name="zoo" android:value="@string/kangaroo" />
另一个方面，使用resource属性会给zoo分配一个数字资源ID，而不是保存在资源中的值。例如：

    <meta-data android:name="zoo" android:resource="@string/kangaroo" />
