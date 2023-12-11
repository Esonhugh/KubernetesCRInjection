## Origin of Injection

When Kubernetes CRD controller which is responsible to analyzing a specific CR is vulnerable, attackers may control certain custom resources and inject malicious payloads, which could trigger malicious behaviors when the controller parses, processes, stores the CRs, or generates other related resources.

## Injection Classification

Injections can be classified via 3 different way.

### By injection points

1. Template injection
2. Delimiter injection e.g. `;` `\n` `}` `"`
3. Object or object reference injection

### By injection results

1. Information disclosure - introducing sensitive environmental variables or injecting into controllable environments
2. Expression injection - executable scripts, expressions, hooks, macros, SQL
3. Configuration file injection
4. Possible file paths, request parameters, database access control bypass
5. Cascading injection (injecting the upper-level CR then generating malicious lower-level components like Pods when creating various CR components)

### By injection points

1. Annotation injection
2. Spec injection due to insufficient validation
3. Controllable references

## Case Study 

### jqctf2023 - Escape? Escaping translation! 

1. The injection point is the lb-type annotation. 
2. The injection type is newline injection / template injection and 
3. The injected content is a malicious pod configuration. 
4. The PoC involves injecting the entire Pod configuration after injecting a name in the annotation, and then injecting flag information present in the secret.
5. It's same as CVE-2022-21701 istio https://paper.seebug.org/1882/ 

### Kubernetes - Nginx ingress controller CVE

1.	The injection point is the annotation.
2.	The injection type is delimiter injection (;).
3.	The injected content can be an nginx configuration file or a Lua script that can leak service account information.
4. Reference: https://hackerone.com/reports/1728174 (Lua expression injection exec) CVE-2021-25742 and CVE-2021-25746. 
5. https://hackerone.com/reports/1378175 (nginx controller file leak)
6. CVE-2023-5044 https://raesene.github.io/blog/2023/10/29/exploiting-CVE-2023-5044/ lua injection exec 

### An undisclosed vulnerability possible CVE unpublished vulnerability

1. Background: In a multi-tenant system, one tenant is hacker. 
2. The injection point is the env environment variable (controlled by tenants). 
3. The injection type is object injection with unsafe ref. 
4. Impact: A single tenant can directly leak either the controller or any specified secrets/configmap key under the controller. 
5. Injecting env using secretKeyRef to inject other data. Reference: https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-using-secret-data

## Information:

1. CRD kubernetes document: https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/
2. CRD validation kubernetes document: https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#validation
3. OpenApi v3 schema: https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.0.0.md#schemaObject

## Services Prone to Impact

The main cause of these vulnerabilities is primarily due to design issues with CRD controllers. In multi-user or SaaS scenarios, users may have indirect or direct control over certain resources within the cluster (which could be CRD resources or other resources). The controllers for these resources may directly or indirectly concatenate user-controlled data, leading to injection vulnerabilities of this nature and further compromising the security within the cluster.
