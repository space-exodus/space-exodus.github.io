# Рефакторинг Actions под архитектуру ECS (28.05.2025)

## Изменения в Space Exodus

- В `NPCCombatSystem.Abilities.TryUseAction`:
    - Убрали дублирующий код проверки действий. Теперь используется только `ActionValidateEvent` из `ActionSystem`.
    - Отключён автоматический поворот NPC к цели. Если нужно — используйте `ActionComponent.RotateOnUse`.

## Изменения в официальной сборке

1. Все компоненты действий (`InstantActionComponent` и др.) перенесены из `Content.Server` в `Content.Shared`.
2. `BaseActionComponent` и `BaseTargetActionComponent` **удалены**. Вместо них теперь отдельные ECS-компоненты.<br>
    Пример замены:<br>
    Было: `if (action is BaseTargetActionComponent target)`<br>
    Стало: `if (TryComp<TargetActionComponent>(actionId))`<br>
3. Все `BaseActionComponent` заменены на `ActionComponent`. Больше кастов типов не выполняется.
4. Для изменения `ActionComponent` используйте `ActionsSystem` вместо прямого доступа к полям.
5. API переработано — теперь действия гибко настраиваются.
6. Для назначения событий используйте `_actions.SetEvent()` вместо прямого присваивания.
7. Все поля компонентов изменяйте **только через API**.
8. В `mapping_actions.yml`:
    - Убраны `BaseActionComponent` и имя.
    - Теперь указывайте:
        - `action` — для прототипа действия,
        - `entity` — для создания прототипа,
        - `tileId` — для размещения тайла.<br>
9. `EntityWorldTargetActionComponent` заменён двумя компонентами:
    - `EntityTargetActionComponent` (с `Event = null`)
    - `WorldTargetActionComponent`  
    *Когда указаны оба, то `WorldActionEvent.Entity` будет получить non-null сущность.*
10. Устаревшие `TryGetActionData`/`TryResolveActionData` заменены на `GetAction()` -> возвращает `Entity<ActionComponent>?`.
11. Все методы API теперь используют `Entity<T>` вместо пар `(actionId, action)`.
12. В `ActionContainerSystem.RemoveAction`:
    - Теперь принимает `Entity<ActionComponent?>?` вместо отдельных аргументов.

*Остальное API системы не изменилось.*

### Оригинальный текст изменений в официальной сборке
```
   all action related components are now in Content.Shared.Actions
    BaseActionComponent and BaseTargetActionComponent are removed, they use proper ecs instead of inheritance now
    if you were casting use TryComp instead if no better api exists,
    e.g. if (action is BaseTargetActionComponent target) becomes if (TryComp<TargetActionComponent>(actionId)
    replace all uses of BaseActionComponent with ActionComponent and do not cast it.
    ActionComponent has access so you need to use ActionsSystem to modify it instead of setting things directly
    api exists for just about everything e.g. SetIcon SetEvent
    use _actions.SetEvent instead of (previously, casting the base action component) setting the component Event field directly
    any other field setting should use public api
    mapping_actions.yml no longer has a BaseActionComponent serialized and a name, you specify either action to use an action prototype, entity to spawn a prototype or tileId to place a tile.
    any custom mapping actions will need to be trimmed down as a result
    EntityWorldTargetActionComponent and event replaced with using EntityTargetAction and WorldTargetAction, with EntityTargetAction's event set to null
    if you have both it can set WorldActionEvent.Entity to a non-null entity
    legacy TryGetActionData and TryResolveActionData removed in favor of GetAction, which just returns an Entity<ActionComponent>?
    SharedActionsSystem and client ActionsSystem methods all changed to use Entity<T>. Pass in action instead of (actionId, action)
    ActionContainerSystem.RemoveAction uses Entity<ActionComponent?>? instead of separate arguments.
    other API from this system is untouched
```
