# ConfigMaps

1. Create ConfigMap
2. Inject into Pod

## Example ConfigMap in Pods
*config-map.yaml*
```
apiVersion: v1
kind: ConfigMap
metadata:
    name: app-config
data:
    APP_COLOR: blue
    APP_MODE: prod
```

*pod-definition.yaml*
```
apiVersion: v1 
kind: Pod 
metadata:
    name: simple-webapp-color
spec:
    containers:
        - name: simple-webapp-color
          image: simple-webapp-color
          ports:
            - containerPort: 8080
          envFrom:
            - configMapRef:
                name: app-config
```

## **2 ways** to Create ConfigMaps:
### 1. Imperative Approach

The imperative approach is used when you want to quickly create a ConfigMap from the command line without writing a YAML file. It’s ideal for testing or simple use cases.

```kubectl create configmap {config-name} --from-literal={key}={value}```
What it does:
* Creates a ConfigMap with key-value pairs directly from the command line.
* Each --from-literal flag adds one key-value entry.

#### For example:
```kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=pod```

Explanation:
* app-config -> name of the ConfigMap
* APP_COLOR=blue -> stored as a key-value pair
* APP_MODE=pod -> another configuration entry
* Result -> a ConfigMap with 2 entries accessible by Pods


### From a file:
```kubectl create configmap <config-name> --from-file={path-to-file}```
What it does:
* Creates a ConfigMap using the contents of a file.
* The file name becomes the key, and its content becomes the value.

```kubectl create  configmap app-config --from-file=app_config.properties```

Explanation:
* ```app_config.properties``` might contain:
```
APP_COLOR=blue
APP_MODE=prod
```
* Kubernetes stores this file inside the ConfigMap.
* Useful for large configs or structured data.

### 2. Declarative Approach
The declarative approach uses YAML files and is the recommended method for production because it is version-controlled, reusable, and easier to manage.

#### Step 1: Define in YAML
*config-map.yaml*
```
apiVersion: v1
kind: ConfigMap
metadata:
    name: app-config
data:
    APP_COLOR: blue
    APP_MODE: prod
```

Explanation:
* apiVersion: v1 -> standard API for ConfigMaps
* kind: ConfigMap -> defines the resource type
* metadata.name -> unique name of the ConfigMap
* data -> key-value pairs used by applications


#### Step 2: Apply the configuration

```kubectl create -f config-map.yaml```

What it does:
* Reads the YAML file and creates the ConfigMap in the cluster.

*Important note*:
* Use kubectl apply -f instead of create if you want to update the ConfigMap later.



# Commands
## List all ConfigMaps in the current namespace, showing their names and basic details:
```kubectl get configmaps```

or

```kubectl get cm```
## Provide detailed information about each ConfigMap, including its key-value data and metadata:
```kubectl describe configmaps```