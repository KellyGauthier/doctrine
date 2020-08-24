# Extensions Doctrine dans API Platform

Une extension dans Doctrine est une classe qui implémente une ou plusieurs interfaces. C'est à dire un contrat d'implémentation qui définit des méthodes vides.

Dans le cadre d'API Platform, cela permet d'étendre les requêtes sur des éléments ou des collections.
Pour cela, une extension implémente nécessairement un **QueryItemExtensionInterface** ou un **QueryCollectionExtensionInterface** ou les deux. Tout dépend de notre besoin.
On peut ajouter une fonction qui définit les conditions d'exécution de l'extension, ainsi que les requêtes à effectuer grâce au _queryBuilder_.

### Exemple d'extension dans API Platform
Source : [Documentation API Platform](https://api-platform.com/docs/core/extensions/)
```
<?php
// api/src/Doctrine/CurrentUserExtension.php

namespace App\Doctrine;

use ApiPlatform\Core\Bridge\Doctrine\Orm\Extension\QueryCollectionExtensionInterface;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Extension\QueryItemExtensionInterface;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use App\Entity\Offer;
use Doctrine\ORM\QueryBuilder;
use Symfony\Component\Security\Core\Security;

final class CurrentUserExtension implements QueryCollectionExtensionInterface, QueryItemExtensionInterface
{
    private $security;

    public function __construct(Security $security)
    {
        $this->security = $security;
    }

    public function applyToCollection(QueryBuilder $queryBuilder, QueryNameGeneratorInterface $queryNameGenerator, string $resourceClass, string $operationName = null): void
    {
        $this->addWhere($queryBuilder, $resourceClass);
    }

    public function applyToItem(QueryBuilder $queryBuilder, QueryNameGeneratorInterface $queryNameGenerator, string $resourceClass, array $identifiers, string $operationName = null, array $context = []): void
    {
        $this->addWhere($queryBuilder, $resourceClass);
    }

    private function addWhere(QueryBuilder $queryBuilder, string $resourceClass): void
    {
        if (Offer::class !== $resourceClass || $this->security->isGranted('ROLE_ADMIN') || null === $user = $this->security->getUser()) {
            return;
        }

        $rootAlias = $queryBuilder->getRootAliases()[0];
        $queryBuilder->andWhere(sprintf('%s.user = :current_user', $rootAlias));
        $queryBuilder->setParameter('current_user', $user->getId());
    }
}
```

L'extension Doctrine s'appuie également sur l'autoconfiguration qui se trouve dans le fichier _service.yaml_. L'autoconfiguration enregistre tous les services automatiquement.

# Le polymorphisme

Le polymorphisme est le fait de pouvoir implémenter des interfaces. Et chaque implémentation peut être personnalisée.

# Les interfaces

Une interface est une classe qui va définir un plan, c'est à dire des signatures de méthodes. Elle sera ensuite implémentée. 
Les classes implémentants des interfaces doivent forcément impléménter les méthodes définies dans l'interface.