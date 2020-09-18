# Instrumenting scripts<a name="scorekeep-scripts"></a>

You can also instrument code that isn't part of your application\. When the X\-Ray daemon is running, it will relay any segments that it receives to X\-Ray, even if they are not generated by the X\-Ray SDK\. Scorekeep uses its own scripts to instrument the build that compiles the application during deployment\.

**Example [https://github.com/awslabs/eb-java-scorekeep/tree/xray/bin/build.sh](https://github.com/awslabs/eb-java-scorekeep/tree/xray/bin/build.sh) – Instrumented build script**  

```
SEGMENT=$(python bin/xray_start.py)
gradle build --quiet --stacktrace &> /var/log/gradle.log; GRADLE_RETURN=$?
if (( GRADLE_RETURN != 0 )); then 
  echo "Gradle failed with exit status $GRADLE_RETURN" >&2
  python bin/xray_error.py "$SEGMENT" "$(cat /var/log/gradle.log)"
  exit 1
fi
python bin/xray_success.py "$SEGMENT"
```

[https://github.com/awslabs/eb-java-scorekeep/tree/xray/bin/xray_start.py](https://github.com/awslabs/eb-java-scorekeep/tree/xray/bin/xray_start.py), [https://github.com/awslabs/eb-java-scorekeep/tree/xray/bin/xray_error.py](https://github.com/awslabs/eb-java-scorekeep/tree/xray/bin/xray_error.py) and [https://github.com/awslabs/eb-java-scorekeep/tree/xray/bin/xray_success.py](https://github.com/awslabs/eb-java-scorekeep/tree/xray/bin/xray_success.py) are simple Python scripts that construct segment objects, convert them to JSON documents, and send them to the daemon over UDP\. If the Gradle build fails, you can find the error message by clicking on the **scorekeep\-build** node in the X\-Ray console service map\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-builderror.png)

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-timeline-builderror.png)

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-exception-builderror.png)