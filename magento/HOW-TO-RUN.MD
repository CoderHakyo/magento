- Adicionar as suas chaves no arquivo magento/auth.json
  - Visite o site: https://commercemarketplace.adobe.com/customer/accessKeys/

- Execute o `docker-compose up -d`

- Em seguida rode os comandos abaixo fora do container docker para configurar o
  nginx corretamente:
  - `docker cp nginx.conf $(docker-compose ps -q web):/etc/nginx/`
  - `docker cp magento.conf $(docker-compose ps -q web):/etc/nginx/conf.d/`

- Em outro terminal pode acompanhar os logs do container web com o comando
  abaixo:
  - `docker logs -f $(docker-compose ps -q web)`

- Depois disso dentro do container execute um restart no nginx:
  - `nginx -t && nginx -s reload`

- Executar o comando abaixo dentro do container web:
  - `composer install`

  - `chown -R application:application var/ && chmod -R 755  var/`

  - ```
    php bin/magento setup:install \
      --base-url=http://localhost:8080 \
      --db-host=db \
      --db-name=magento \
      --db-user=magento \
      --db-password=magentopass \
      --admin-firstname=Admin \
      --admin-lastname=User \
      --admin-email=admin@example.com \
      --admin-user=admin \
      --admin-password=admin123 \
      --language=pt_BR \
      --currency=BRL \
      --timezone=America/Sao_Paulo \
      --use-rewrites=1 \
      --search-engine=elasticsearch8 \
      --elasticsearch-host=elasticsearch \
      --elasticsearch-port=9200
      ```

- Ao final ao visitar `http://localhost:8080` vai perceber que a pagina carrega
  porem sem estilo nenhum, para corrigir esse problema vamos seguir os passos
abaixo:

- Executar o comando abaixo para gerar os "assets" nos idiomas pt-BR e en-US:
  - `php bin/magento setup:static-content:deploy -f en_US pt_BR`

- Execute os comandos abaixo para mudar as permissoes:
  - `chmod -R 775 pub/static generated`
  - `chown -R application:application pub/static generated`

- Caso queira criar um usuario admin execute o comando abaixo:
  - `php bin/magento admin:user:create --admin-user=admin2 --admin-password='Magento@1234' --admin-email=admin2@example.com --admin-firstname=Admin --admin-lastname=User`


- OS PASSOS ABAIXO SAO EXECUTADOS DENTRO DO CONTAINER WEB DO DOCKER

  - BloomReach Discovery (https://github.com/bloomreach/discovery-connector-magento) - PAGO
    - `composer require bloomreach/bloomreach-connector-magento`
    - `php bin/magento module:enable Bloomreach_Connector`
    - `php bin/magento setup:upgrade`
    - `php bin/magento setup:di:compile`
    - `php bin/magento cache:flush`

    - Com isso vai quebrar as permissoes pois instalamos uma nova dependencia,
      portanto precisamos atualizar as permissoes:
      - Caso quebre o CSS rode novamente o comando abaixo:
        - `php bin/magento setup:static-content:deploy -f en_US pt_BR`

      - `chown -R application:application .`
      - `find var generated pub/static -type d -exec chmod 775 {} \;`
      - `find var generated pub/static -type f -exec chmod 664 {} \;`

    - Apos ajustar as permissoes e arrumar o css no painel de admin, no menu do
      lado esquerdo encontre o botao `store`, dentro do submenu havera
      `configuration` e dentro dessa tela agora tera a opcao `Bloomreach Discovery`.

  - Algolia (https://github.com/algolia/algoliasearch-magento-2) - TEM VERSAO FREE POREM LIMITADA A NUMERO DE REGISTROS / BUSCAS
    - `composer require algolia/algoliasearch-magento-2`
    - `bin/magento module:enable Algolia_AlgoliaSearch`
    - `bin/magento setup:upgrade`
    - `bin/magento setup:di:compile`
    - `php bin/magento cache:flush`

    - Com isso vamos ter problemas de permissao portanto rode os comandos abaixo:
      - Caso quebre o CSS rode novamente o comando abaixo:
        - `php bin/magento setup:static-content:deploy -f en_US pt_BR`

      - `chown -R application:application .`
      - `find var generated pub/static -type d -exec chmod 775 {} \;`
      - `find var generated pub/static -type f -exec chmod 664 {} \;`

    - Visite a pagina de admin e no menu lateral esquerda encontre o botao `Store`
      e no submenu havera a opcao `Configuration` e agora tera a opcao `Algolia
      Search` dentro da pagina

    - Apos a instalacao do Algolia podemos seguir o tutorial desse video, que
      ja teremos o Algolia minimamente funcionando, para validar isso podemos
      efetuar uma busca no campo search na home aparecera o logo do Algolia.
      - https://www.youtube.com/watch?v=twEj_VBWxp8
      - No video ele pede para executar o seguinte o comando:
        - `bin/magento indexer:reindex algolia_products algolia_categories algolia_pages algolia_suggestions algolia_additional_sections`
      - Apos criar um novo produto ele nao aparece de cara, precisa efetuar um
        reindex no elasticsearch;
        - `php bin/magento indexer:reindex`
      - Caso venha a quebrar alguma coisa ao visitar a pagina rode o comando
        abaixo:
        - `php bin/magento setup:di:compile`


