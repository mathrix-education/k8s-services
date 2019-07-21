FROM verdaccio/verdaccio:4

USER root

# Build verdaccio-github-oauth-ui#verdaccio-4
RUN apk add --no-cache bash wget unzip \
	&& wget https://github.com/n4bb12/verdaccio-github-oauth-ui/archive/verdaccio-4.zip -O verdaccio-github-oauth-ui.zip \
	&& unzip verdaccio-github-oauth-ui.zip \
	&& cd verdaccio-github-oauth-ui-verdaccio-4 \
	&& yarn install && yarn run build \
	&& cd $(printenv VERDACCIO_APPDIR) \
	&& yarn add file:/$(printenv VERDACCIO_APPDIR)/verdaccio-github-oauth-ui-verdaccio-4

# Cleanup
RUN apk del bash wget unzip \
	&& rm -rf verdaccio-github-oauth-ui-verdaccio-4

ENV NODE_ENV=production

USER verdaccio