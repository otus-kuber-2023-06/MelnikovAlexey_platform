## API

### Helm Chart Repository
- `GET /index.yaml` - retrieved when you run `helm repo add chartmuseum http://localhost:8080/`
- `GET /charts/mychart-0.1.0.tgz` - retrieved when you run `helm install chartmuseum/mychart`
- `GET /charts/mychart-0.1.0.tgz.prov` - retrieved when you run `helm install` with the `--verify` flag

### Chart Manipulation
- `POST /api/charts` - upload a new chart version
- `POST /api/prov` - upload a new provenance file
- `DELETE /api/charts/<name>/<version>` - delete a chart version (and corresponding provenance file)
- `GET /api/charts` - list all charts
- `GET /api/charts/<name>` - list all versions of a chart
- `GET /api/charts/<name>/<version>` - describe a chart version
- `GET /api/charts/<name>/<version>/templates` - get chart template
- `GET /api/charts/<name>/<version>/values` - get chart values
- `HEAD /api/charts/<name>` - check if chart exists (any versions)
- `HEAD /api/charts/<name>/<version>` - check if chart version exists

### Server Info
- `GET /` - HTML welcome page
- `GET /info` - returns current ChartMuseum version
- `GET /health` - returns 200 OK

## Uploading a Chart Package
<sub>*Follow **"How to Run"** section below to get ChartMuseum up and running at ht<span>tp:/</span>/localhost:8080*<sub>

First create `mychart-0.1.0.tgz` using the [Helm CLI](https://docs.helm.sh/using_helm/#installing-helm):
```
cd mychart/
helm package .
```

Upload `mychart-0.1.0.tgz`:
```bash
curl --data-binary "@mychart-0.1.0.tgz" http://localhost:8080/api/charts
```

If you've signed your package and generated a [provenance file](https://github.com/helm/helm-www/blob/master/content/en/docs/topics/provenance.md), upload it with:
```bash
curl --data-binary "@mychart-0.1.0.tgz.prov" http://localhost:8080/api/prov
```

Both files can also be uploaded at once (or one at a time) on the `/api/charts` route using the `multipart/form-data` format:

```bash
curl -F "chart=@mychart-0.1.0.tgz" -F "prov=@mychart-0.1.0.tgz.prov" http://localhost:8080/api/charts
```

You can also use the [helm-push plugin](https://github.com/chartmuseum/helm-push):
```
helm cm-push mychart/ chartmuseum
```

## Installing Charts into Kubernetes
Add the URL to your *ChartMuseum* installation to the local repository list:
```bash
helm repo add chartmuseum http://localhost:8080
```

Search for charts:
```bash
helm search repo chartmuseum/
```

Install chart:
```bash
helm install chartmuseum/mychart --generate-name
```