# k8s-opencart-installation

## Installation steps

```
export REPO_FOLDER="$HOME/work/git/opencart"
export CHART_NAME='opencart-test'

export OPEN_CART_HOST=example.com
export OPEN_CART_APP_USER=admin
export OPEN_CART_APP_PASSWORD=admin
export OPEN_CART_DATABASE_ROOT_PASSWORD=adminrootpass
export OPEN_CART_DATABASE_PASSWORD=opencartdbpass

export CHART_VERSION='12.4.2'
export CHART_FOLDER='opencart-chart'

function replace_in_file(){
    local placeHolder=$1
    local value=$2

    local ESCAPED_KEYWORD=$(printf '%s\n' "$placeHolder" | sed -e 's/[]\/$*.^[]/\\&/g');
    local ESCAPED_REPLACE=$(printf '%s\n' "$value" | sed -e 's/[\/&]/\\&/g')

    local filePath="$3"

    echo "Will replace $ESCAPED_KEYWORD with $ESCAPED_REPLACE in file $filePath"
    sed -i '' -e "s/$ESCAPED_KEYWORD/$ESCAPED_REPLACE/g" $filePath
}

cd $REPO_FOLDER

rm -rf opencart-chart
mkdir opencart-chart

helm repo add bitnami https://charts.bitnami.com/bitnami
helm search repo opencart --versions

helm pull bitnami/opencart -d "$CHART_FOLDER" --version $CHART_VERSION

cd "$CHART_FOLDER"
tar -xvzf opencart-$CHART_VERSION.tgz

kubectl delete ingress "$CHART_NAME"
helm uninstall $CHART_NAME --wait 

helm install $CHART_NAME ./opencart --set opencartHost=$OPEN_CART_HOST,opencartUsername=$OPEN_CART_APP_USER,opencartPassword=$OPEN_CART_APP_PASSWORD,mariadb.auth.rootPassword=$OPEN_CART_DATABASE_ROOT_PASSWORD,mariadb.auth.password=$OPEN_CART_DATABASE_PASSWORD,opencartEnableHttps=true,service.type=ClusterIP

replace_in_file "<chart_name>" "$CHART_NAME" ../opencart-ingress.yaml
replace_in_file "<open_cart_host>" "$OPEN_CART_HOST" ../opencart-ingress.yaml
kubectl apply -f ../opencart-ingress.yaml

echo ---------------------------------------------
echo OpenCart URL: https://$OPEN_CART_HOST
echo OpenCart Admin URL: https://$OPEN_CART_HOST/admin
echo OpenCart user: $OPEN_CART_APP_USER
echo OpenCart pass: $OPEN_CART_APP_PASSWORD
echo ---------------------------------------------

echo Execute the following command to watch for OpenCart startup: watch -n 10 curl https://$OPEN_CART_HOST

```

## Links
- [OpenCart official page](https://www.opencart.com/)
- [OpenCart helm chart](https://github.com/bitnami/charts/tree/main/bitnami/opencart)
