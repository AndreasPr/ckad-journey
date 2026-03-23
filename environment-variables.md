# Environment Variables

## Env Value Types
1. Plain Key Value
2. ConfigMap
3. Secrets

```docker run -e APP_COLOR=red simple-webapp```

## Example 'pod-definition.yaml' for *Plain Key Value*
```
apiVersion: v1
kind: Pod
metadata:
    name: simple-webapp
spec:
    containers:
    - name: simple-webapp
        image: simple-webapp
        ports:
            - containerPort: 8080
        env:
            - name: APP_COLOR
              value: red
```

## Example 'pod-definition.yaml' for *ConfigMap*
```
env:
    - name: APP_COLOR
      valueFrom:
          configMapKeyRef:
```

## Example 'pod-definition.yaml' for *Secrets*
```
env:
    - name: APP_COLOR
      valueFrom:
          secretKeyRef:
```


