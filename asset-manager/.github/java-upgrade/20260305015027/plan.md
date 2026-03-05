# Upgrade Plan: asset-manager (20260305015027)

- **Generated**: 2026-03-05 01:50:27 UTC
- **HEAD Branch**: main
- **HEAD Commit ID**: c5b9fb9e91a124c0bc969d90946c521281115f7d

## Available Tools

**JDKs**
- JDK 1.8.0_482: /usr/local/sdkman/candidates/java/8.0.482-tem (current project JDK, used by Step 2)
- JDK 17: **<TO_BE_INSTALLED>** (needed for Step 3 - Spring Boot 3.2 migration)
- JDK 21: **<TO_BE_INSTALLED>** (needed for Step 5 and Step 6 - target version)

**Build Tools**
- Maven Wrapper (mvnw/mvnw.cmd): Available in project root (preferred)

## Guidelines

> Note: You can add any specific guidelines or constraints for the upgrade process here if needed, bullet points are preferred.

## Options

- Working branch: appmod/java-upgrade-20260305015027
- Run tests before and after the upgrade: true

## Upgrade Goals

- Upgrade Java from 8 to 21 LTS

### Technology Stack

| Technology/Dependency | Current | Min Compatible | Why Incompatible |
| --------------------- | ------- | -------------- | ---------------- |
| Java                  | 8       | 21             | User requested target version |
| Spring Boot           | 2.7.18  | 3.2.0          | Spring Boot 2.7.x supports Java 8-17 only; Java 21 requires Spring Boot 3.2+ |
| Spring Framework      | 5.3.x   | 6.1.x          | Spring Boot 3.2 requires Spring Framework 6.1+ |
| Hibernate ORM         | 5.6.x   | 6.2.x          | Spring Boot 3.x requires Hibernate 6.x for JPA support |
| javax.persistence ⚠️ EOL | 2.2   | N/A            | Replaced by jakarta.persistence in Spring Boot 3.x; javax namespace EOL |
| javax.annotation ⚠️ EOL | 1.3.2 | N/A            | Replaced by jakarta.annotation in Spring Boot 3.x; javax namespace EOL |
| javax.servlet ⚠️ EOL  | 4.0.1   | N/A            | Replaced by jakarta.servlet in Spring Boot 3.x; javax namespace EOL |
| AWS SDK S3            | 2.25.13 | 2.25.13        | - |
| PostgreSQL Driver     | 42.5.x  | 42.5.x         | - |
| Lombok                | 1.18.24 | 1.18.24        | - |
| Thymeleaf             | 3.0.x   | 3.1.x          | Spring Boot 3.x upgrades Thymeleaf to 3.1+ |
| Spring AMQP           | 2.4.x   | 3.0.x          | Spring Boot 3.x upgrades Spring AMQP to 3.0+ |
| Jackson               | 2.13.x  | 2.15.x         | Spring Boot 3.x upgrades Jackson to 2.15+ |

### Derived Upgrades

- **Upgrade Spring Boot from 2.7.18 to 3.2.x**: Spring Boot 2.7.x maximum Java support is 17; Java 21 requires Spring Boot 3.2+ which has Java 21 LTS support.
- **Upgrade Spring Framework from 5.3.x to 6.1.x**: Spring Boot 3.2 requires Spring Framework 6.1+. This upgrade is automatically managed through Spring Boot BOM.
- **Upgrade Hibernate ORM from 5.6.x to 6.2.x**: Spring Boot 3.x requires Hibernate 6.x for JPA/persistence support. This upgrade is automatically managed through Spring Boot BOM.
- **Migrate from javax.* to jakarta.* namespace**: Spring Boot 3.x uses Jakarta EE 9+ which replaces javax.* with jakarta.* namespace. Required changes:
  - `javax.persistence.*` → `jakarta.persistence.*` (JPA annotations in web and worker modules)
  - `javax.annotation.PostConstruct` → `jakarta.annotation.PostConstruct` (used in service classes)
  - `javax.servlet.*` → `jakarta.servlet.*` (used in web module configuration)
- **Upgrade Thymeleaf to 3.1.x**: Spring Boot 3.x upgrades Thymeleaf from 3.0.x to 3.1.x for Jakarta EE compatibility. This upgrade is automatically managed through Spring Boot BOM.
- **Upgrade Spring AMQP to 3.0.x**: Spring Boot 3.x upgrades Spring AMQP from 2.4.x to 3.0.x. This upgrade is automatically managed through Spring Boot BOM.
- **Upgrade Jackson to 2.15.x**: Spring Boot 3.x upgrades Jackson from 2.13.x to 2.15.x. This upgrade is automatically managed through Spring Boot BOM.

## Upgrade Steps

- **Step 1: Setup Environment**
  - **Rationale**: Install JDK 17 and JDK 21 required for the upgrade process. JDK 17 serves as an intermediate bridge for Spring Boot 3.2 migration, while JDK 21 is the final target version.
  - **Changes to Make**:
    - [ ] Install JDK 17 using `#install_jdk version=17`
    - [ ] Install JDK 21 using `#install_jdk version=21`
    - [ ] Verify Maven Wrapper is executable (`chmod +x mvnw` if needed)
  - **Verification**:
    - Command: `#list_jdks`
    - Expected: JDK 17 and JDK 21 available at specified paths

---

- **Step 2: Setup Baseline**
  - **Rationale**: Establish pre-upgrade compilation and test results with current JDK 8. This baseline forms the acceptance criteria for the upgrade (compilation must remain successful, test pass rate must be maintained or improved).
  - **Changes to Make**:
    - [ ] Clean build and compile all modules with current JDK 8
    - [ ] Run full test suite with current JDK 8
    - [ ] Document compilation status, test pass count, and any existing failures
  - **Verification**:
    - Command: `./mvnw clean test -q`
    - JDK: /usr/local/sdkman/candidates/java/8.0.482-tem
    - Expected: Document SUCCESS/FAILURE for compilation and test execution (establishes baseline metrics)

---

- **Step 3: Upgrade to Spring Boot 3.2.x with Jakarta EE Migration**
  - **Rationale**: Spring Boot 3.2.x is required for Java 21 support and brings Spring Framework 6.1.x + Hibernate 6.2.x automatically. Using JDK 17 for this step provides a safer intermediate bridge before jumping to JDK 21. Jakarta namespace migration (javax.* → jakarta.*) must happen at this step as Spring Boot 3.x requires Jakarta EE 9+.
  - **Changes to Make**:
    - [ ] Update `spring-boot-starter-parent` version from 2.7.18 to 3.2.3 in root pom.xml and module pom.xml files
    - [ ] Migrate all javax.* imports to jakarta.* namespace across both web and worker modules:
      - `javax.persistence.*` → `jakarta.persistence.*` (Entity, Table, Id, GeneratedValue, Column, etc.)
      - `javax.annotation.PostConstruct` → `jakarta.annotation.PostConstruct`
      - `javax.servlet.*` → `jakarta.servlet.*` (if used in web configuration)
    - [ ] Update any explicit Spring Framework version declarations (if present) to 6.1.x
    - [ ] Fix any compilation errors related to API changes in Spring Boot 3.x
  - **Verification**:
    - Command: `./mvnw clean test-compile -q`
    - JDK: [JDK 17 path from Step 1]
    - Expected: Compilation SUCCESS (tests may fail at this intermediate stage)

---

- **Step 4: Update Java Source and Target Version to 17**
  - **Rationale**: After successfully migrating to Spring Boot 3.2.x on JDK 17, update the Maven compiler source/target to Java 17 to ensure code is compiled with Java 17 language features and bytecode level.
  - **Changes to Make**:
    - [ ] Update `<java.version>17</java.version>` in root pom.xml properties section
    - [ ] Remove explicit `<source>` and `<target>` from maven-compiler-plugin if present (Spring Boot manages these via java.version property)
    - [ ] Verify compilation with updated Java 17 bytecode level
  - **Verification**:
    - Command: `./mvnw clean test-compile -q`
    - JDK: [JDK 17 path from Step 1]
    - Expected: Compilation SUCCESS with Java 17 bytecode

---

- **Step 5: Upgrade to Java 21**
  - **Rationale**: With Spring Boot 3.2.x successfully running on Java 17, upgrade to the final target Java 21 LTS. Spring Boot 3.2.x fully supports Java 21, so this step focuses on updating the java.version property and verifying compilation.
  - **Changes to Make**:
    - [ ] Update `<java.version>21</java.version>` in root pom.xml properties section
    - [ ] Verify compilation with JDK 21
    - [ ] Check for any Java 21-specific warnings or deprecations in compiler output
  - **Verification**:
    - Command: `./mvnw clean test-compile -q`
    - JDK: [JDK 21 path from Step 1]
    - Expected: Compilation SUCCESS with Java 21 bytecode

---

- **Step 6: Final Validation**
  - **Rationale**: Verify all upgrade goals are met: Java 21 active, Spring Boot 3.2.x active, project compiles successfully, and 100% of tests pass (matching or exceeding baseline from Step 2).
  - **Changes to Make**:
    - [ ] Verify all target versions are set correctly in pom.xml files (Spring Boot 3.2.x, Java 21)
    - [ ] Resolve any remaining TODOs or temporary workarounds from previous steps
    - [ ] Run full clean build and test suite with JDK 21
    - [ ] Fix any test failures through iterative debugging until 100% pass rate achieved
    - [ ] Compare final test results with baseline from Step 2 (must match or exceed)
  - **Verification**:
    - Command: `./mvnw clean test -q`
    - JDK: [JDK 21 path from Step 1]
    - Expected: Compilation SUCCESS + 100% test pass (all tests from baseline must pass)

## Key Challenges

- **Jakarta EE Namespace Migration (javax.* → jakarta.*)**
  - **Challenge**: The entire codebase uses javax.* packages (persistence, annotation, servlet) that must migrate to jakarta.* in Spring Boot 3.x. This affects both web and worker modules with JPA entities, service classes (@PostConstruct), and web configurations.
  - **Strategy**: Perform bulk namespace migration in Step 3 using find-and-replace across all Java source files. Key imports to update: `javax.persistence.*`, `javax.annotation.PostConstruct`, `javax.servlet.*`. Spring Boot 3.2.x on JDK 17 provides a stable environment to verify the migration before moving to Java 21.

- **Hibernate 5.6.x to 6.2.x Major Version Jump**
  - **Challenge**: Hibernate 6.x introduces breaking changes including updated query syntax, behavior changes in entity lifecycle callbacks, and stricter validation rules. The upgrade happens automatically via Spring Boot BOM but may cause runtime issues.
  - **Strategy**: After Spring Boot 3.2.x migration in Step 3, carefully test all JPA repository methods and custom queries. Watch for Hibernate-specific annotations (e.g., @Type, @Formula) that may need updates. Address schema validation warnings and deprecated API usage.

- **Multi-Module Maven Project Coordination**
  - **Challenge**: Project has two modules (web and worker) that must be upgraded in lockstep. Both modules share dependencies and must remain compatible throughout the upgrade process.
  - **Strategy**: Upgrade Spring Boot version in the root pom.xml first (affects both modules via parent), then apply Jakarta namespace migration to both modules simultaneously in Step 3. Verify compilation of both modules together to catch any cross-module breaking changes early.

- **Spring Boot 2.7 to 3.2 Breaking Changes**
  - **Challenge**: Spring Boot 3.x introduces numerous breaking changes beyond Jakarta namespace including property changes (application.properties), autoconfiguration changes, and removed deprecated APIs.
  - **Strategy**: Leverage Spring Boot 3.2.x's comprehensive migration guide. Key areas to check: application.properties syntax (spring.jpa.hibernate.* properties), security configuration (if used), and actuator endpoints. Use JDK 17 in Step 3 to isolate Spring Boot migration issues from Java version issues.

- **Test Compatibility During Intermediate Steps**
  - **Challenge**: Tests may fail during intermediate steps (Steps 3-5) due to framework changes, even though compilation succeeds. This is expected but must not block progress.
  - **Strategy**: Focus on compilation success in Steps 3-5, allowing test failures. Reserve test fixing for Step 6 (Final Validation) where we have the complete upgrade in place. Document any expected test failures during intermediate steps to avoid confusion.

## Plan Review

### Completeness Check

✅ **All sections fully populated**:
- Available Tools: Complete with TO_BE_INSTALLED markers for JDK 17 and JDK 21
- Guidelines: Section present with user note
- Options: Working branch and test execution settings confirmed
- Upgrade Goals: Primary goal clearly stated (Java 8 → 21)
- Technology Stack: Comprehensive table with 12 dependencies analyzed, EOL dependencies properly marked (javax.* packages)
- Derived Upgrades: 7 derived upgrades identified with clear justifications
- Upgrade Steps: 6 steps following mandatory structure (Setup Environment, Setup Baseline, 3 upgrade steps, Final Validation)
- Key Challenges: 5 major challenges identified with mitigation strategies

✅ **No placeholders remaining**: All required sections filled with actual project data (except user-configurable Guidelines note)

✅ **Upgrade steps structure verified**:
- Step 1: Setup Environment (mandatory) ✓
- Step 2: Setup Baseline (mandatory) ✓
- Steps 3-5: Progressive upgrades (Spring Boot 3.2 + Jakarta, Java 17 compiler, Java 21) ✓
- Step 6: Final Validation (mandatory) ✓

✅ **All HTML comments removed**: Planning guidance comments cleaned from final document

### Feasibility Assessment

**Overall Assessment**: ✅ **FEASIBLE** - The upgrade path is realistic and achievable with manageable risks.

**Strengths**:
1. **Sound intermediate strategy**: Using JDK 17 as a bridge before jumping to JDK 21 reduces risk and isolates issues
2. **Correct dependency order**: Spring Boot upgrade drives all framework upgrades (Spring Framework 6.1, Hibernate 6.2) automatically via BOM
3. **Early namespace migration**: Jakarta migration happens in Step 3 alongside Spring Boot upgrade, which is the correct timing
4. **Multi-module awareness**: Plan explicitly addresses both web and worker modules throughout
5. **Test strategy**: Realistic expectation that tests may fail during intermediate steps, with focused fixing in Final Validation

**Identified Risks and Mitigations**:
1. **Hibernate 6.x breaking changes**: 
   - Risk: Major version upgrade may affect query syntax, entity callbacks, validation rules
   - Mitigation: Test all JPA repositories in Step 3, watch for @Type/@Formula annotations, address schema validation
   - Impact: MEDIUM (manageable through iterative testing)

2. **Jakarta namespace scope**: 
   - Risk: Unknown number of javax.* usages across both modules
   - Mitigation: Bulk find-and-replace strategy identified for key imports
   - Impact: LOW (mechanical change, easily verifiable by compilation)

3. **Spring Boot 2.7 → 3.2 configuration changes**:
   - Risk: application.properties syntax changes, autoconfiguration differences
   - Mitigation: Leverage Spring Boot migration guide, focus on known areas (spring.jpa.hibernate.*, security, actuator)
   - Impact: MEDIUM (requires careful testing)

4. **Test failures during intermediate steps**:
   - Risk: May be difficult to distinguish expected vs. unexpected failures
   - Mitigation: Baseline results from Step 2 provide clear acceptance criteria
   - Impact: LOW (expected behavior, Final Validation addresses)

**Known Limitations**:
- Final test pass rate dependent on test quality and coverage (baseline will establish expectations)
- Some Hibernate 6.x compatibility issues may only surface at runtime, requiring iterative debugging
- Spring Boot 3.x property changes may require investigation beyond documented migration paths

**Recommendation**: ✅ **PROCEED** - The plan is well-structured with clear risk mitigation. The 6-step approach balances thoroughness with efficiency. No blocking issues identified.
