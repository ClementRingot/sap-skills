---
name: abap-unit-tests
description: This skill provides templates and rules for generating ABAP Unit tests for RAP Business Objects. It covers CRUD operations, validations, determinations, actions, and test double setup with CDS and OSQL test environments.
---

## Test Class Template

```abap
"! @testing ZR_{NAME}TP
CLASS ltcl_{name} DEFINITION FINAL FOR TESTING
  DURATION SHORT
  RISK LEVEL HARMLESS.

  PRIVATE SECTION.
    CLASS-DATA:
      environment     TYPE REF TO if_cds_test_environment,
      sql_environment TYPE REF TO if_osql_test_environment.

    CLASS-METHODS:
      class_setup,
      class_teardown.

    METHODS:
      setup,
      teardown,

      " CRUD Tests
      create_entity FOR TESTING,
      update_entity FOR TESTING,
      delete_entity FOR TESTING,
      read_entity FOR TESTING,

      " Validation Tests
      validate_required_fields_ok FOR TESTING,
      validate_required_fields_fail FOR TESTING,

      " Determination Tests
      determination_sets_defaults FOR TESTING,

      " Action Tests
      action_approve FOR TESTING.
ENDCLASS.

CLASS ltcl_{name} IMPLEMENTATION.

  METHOD class_setup.
    " Create CDS test double
    environment = cl_cds_test_environment=>create( i_for_entity = 'ZR_{NAME}TP' ).

    " Create OSQL test double for DB table
    sql_environment = cl_osql_test_environment=>create(
      i_dependency_list = VALUE #( ( 'Z{NAME}' ) ) ).
  ENDMETHOD.

  METHOD class_teardown.
    environment->destroy( ).
    sql_environment->destroy( ).
  ENDMETHOD.

  METHOD setup.
    environment->clear_doubles( ).
    sql_environment->clear_doubles( ).
  ENDMETHOD.

  METHOD teardown.
    ROLLBACK ENTITIES.
  ENDMETHOD.

  " === CRUD Tests ===

  METHOD create_entity.
    MODIFY ENTITIES OF zr_{name}tp
      ENTITY {Alias}
      CREATE FIELDS ( Field1 Field2 )
      WITH VALUE #( ( %cid = 'CID1'
                      Field1 = 'Value1'
                      Field2 = 'Value2' ) )
      MAPPED DATA(mapped)
      FAILED DATA(failed)
      REPORTED DATA(reported).

    cl_abap_unit_assert=>assert_initial( failed ).
    cl_abap_unit_assert=>assert_not_initial( mapped ).

    COMMIT ENTITIES RESPONSES
      FAILED DATA(commit_failed)
      REPORTED DATA(commit_reported).

    cl_abap_unit_assert=>assert_initial( commit_failed ).
  ENDMETHOD.

  METHOD update_entity.
    sql_environment->insert_test_data(
      VALUE z{name}( key_field = '001' field_1 = 'Old' ) ).

    MODIFY ENTITIES OF zr_{name}tp
      ENTITY {Alias}
      UPDATE FIELDS ( Field1 )
      WITH VALUE #( ( KeyField = '001'
                      Field1 = 'New' ) )
      FAILED DATA(failed)
      REPORTED DATA(reported).

    cl_abap_unit_assert=>assert_initial( failed ).
  ENDMETHOD.

  METHOD delete_entity.
    sql_environment->insert_test_data(
      VALUE z{name}( key_field = '001' ) ).

    MODIFY ENTITIES OF zr_{name}tp
      ENTITY {Alias}
      DELETE FROM VALUE #( ( KeyField = '001' ) )
      FAILED DATA(failed)
      REPORTED DATA(reported).

    cl_abap_unit_assert=>assert_initial( failed ).
  ENDMETHOD.

  METHOD read_entity.
    sql_environment->insert_test_data(
      VALUE z{name}( key_field = '001' field_1 = 'Test' ) ).

    READ ENTITIES OF zr_{name}tp
      ENTITY {Alias}
      ALL FIELDS
      WITH VALUE #( ( KeyField = '001' ) )
      RESULT DATA(result)
      FAILED DATA(failed).

    cl_abap_unit_assert=>assert_initial( failed ).
    cl_abap_unit_assert=>assert_equals( exp = 1 act = lines( result ) ).
    cl_abap_unit_assert=>assert_equals( exp = 'Test' act = result[ 1 ]-Field1 ).
  ENDMETHOD.

  " === Validation Tests ===

  METHOD validate_required_fields_ok.
    MODIFY ENTITIES OF zr_{name}tp
      ENTITY {Alias}
      CREATE FIELDS ( Field1 Field2 )
      WITH VALUE #( ( %cid = 'CID1'
                      Field1 = 'OK'
                      Field2 = 'OK' ) )
      MAPPED DATA(mapped)
      FAILED DATA(failed)
      REPORTED DATA(reported).

    COMMIT ENTITIES RESPONSES
      FAILED DATA(commit_failed)
      REPORTED DATA(commit_reported).

    cl_abap_unit_assert=>assert_initial( commit_failed ).
  ENDMETHOD.

  METHOD validate_required_fields_fail.
    MODIFY ENTITIES OF zr_{name}tp
      ENTITY {Alias}
      CREATE FIELDS ( Field1 )
      WITH VALUE #( ( %cid = 'CID1'
                      Field1 = 'OK' ) )  " Field2 missing
      MAPPED DATA(mapped)
      FAILED DATA(failed)
      REPORTED DATA(reported).

    COMMIT ENTITIES RESPONSES
      FAILED DATA(commit_failed)
      REPORTED DATA(commit_reported).

    cl_abap_unit_assert=>assert_not_initial( commit_failed ).
  ENDMETHOD.

  " === Determination Tests ===

  METHOD determination_sets_defaults.
    MODIFY ENTITIES OF zr_{name}tp
      ENTITY {Alias}
      CREATE FIELDS ( Field1 )
      WITH VALUE #( ( %cid = 'CID1'
                      Field1 = 'Test' ) )
      MAPPED DATA(mapped)
      FAILED DATA(failed)
      REPORTED DATA(reported).

    READ ENTITIES OF zr_{name}tp
      ENTITY {Alias}
      ALL FIELDS
      WITH CORRESPONDING #( mapped-{alias} )
      RESULT DATA(result).

    " Verify the determination set the default value
    cl_abap_unit_assert=>assert_not_initial( result[ 1 ]-DefaultField ).
  ENDMETHOD.

  " === Action Tests ===

  METHOD action_approve.
    sql_environment->insert_test_data(
      VALUE z{name}( key_field = '001' status = 'DRAFT' ) ).

    MODIFY ENTITIES OF zr_{name}tp
      ENTITY {Alias}
      EXECUTE approve FROM VALUE #( ( KeyField = '001' ) )
      RESULT DATA(result)
      FAILED DATA(failed)
      REPORTED DATA(reported).

    cl_abap_unit_assert=>assert_initial( failed ).

    READ ENTITIES OF zr_{name}tp
      ENTITY {Alias}
      FIELDS ( Status )
      WITH VALUE #( ( KeyField = '001' ) )
      RESULT DATA(read_result).

    cl_abap_unit_assert=>assert_equals(
      exp = 'APPROVED' act = read_result[ 1 ]-Status ).
  ENDMETHOD.

ENDCLASS.
```

---

## Testing Rules

1. **ALWAYS** `ROLLBACK ENTITIES` in `teardown`
2. **ALWAYS** create CDS and SQL test doubles
3. **ALWAYS** test both success AND failure cases
4. **NEVER** depend on real system data
5. Use `COMMIT ENTITIES RESPONSES` to trigger validations in tests

---

## Minimum Coverage Expected

| BDEF Element | Required Test |
|-------------|---------------|
| create | Creation OK + validation failure |
| update | Modification OK |
| delete | Deletion OK |
| validation | Success case + error case per validation |
| determination | Verify the value set by the determination |
| action | Verify state change after action execution |
| feature control | Verify enabled/disabled based on entity state |

---

## Output

Produce the complete test class ready to be inserted in the Behavior Pool
as a local test include `ltcl_{name}` or as a global class `ZTC_{NAME}TP`.
