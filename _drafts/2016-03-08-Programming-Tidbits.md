- ##### Problem 1
  - ###### Issue
      Casting Application super class to its custom subclass i.e ((BaseApplication) getApplication()) will through ClassCastException.
  - ###### Fix
      Need to declared custom Application subclass(BaseApplication) in the AndroidManifest.xml.
      {% highlight xml linenos %}
      <application
           android:name=".BaseApplication"
           android:icon="..."
           android:label="...">
      </application>
      {% endhighlight %}
