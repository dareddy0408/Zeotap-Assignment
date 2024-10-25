# Zeotap-Assinment
import java.util.HashMap;
import java.util.Map;

class Node {
    String nodeType; // "operator" or "operand"
    Node left;      // Left child Node
    Node right;     // Right child Node
    Map<String, String> value; // Optional value for operand nodes

    public Node(String nodeType, Node left, Node right, Map<String, String> value) {
        this.nodeType = nodeType;
        this.left = left;
        this.right = right;
        this.value = value;
    }
}

public class RuleEngine {

    public static Node parseToAST(String ruleString) {
        ruleString = ruleString.trim();
        if (ruleString.contains("AND")) {
            String[] parts = ruleString.split(" AND ");
            return new Node("operator", parseToAST(parts[0]), parseToAST(String.join(" AND ", parts).substring(parts[0].length() + 5)), null);
        } else if (ruleString.contains("OR")) {
            String[] parts = ruleString.split(" OR ");
            return new Node("operator", parseToAST(parts[0]), parseToAST(String.join(" OR ", parts).substring(parts[0].length() + 4)), null);
        } else {
            // Simple format: "(attribute operator value)"
            String[] tokens = ruleString.replaceAll("[()]", "").trim().split(" ");
            if (tokens.length == 3) {
                Map<String, String> operandValue = new HashMap<>();
                operandValue.put("attribute", tokens[0]);
                operandValue.put("operator", tokens[1]);
                operandValue.put("value", tokens[2].replace("\"", ""));
                return new Node("operand", null, null, operandValue);
            }
        }
        throw new IllegalArgumentException("Invalid rule string");
    }

    public static boolean evaluateCondition(String attributeValue, String operator, String value) {
        switch (operator) {
            case ">":
                return Integer.parseInt(attributeValue) > Integer.parseInt(value);
            case "<":
                return Integer.parseInt(attributeValue) < Integer.parseInt(value);
            case "=":
                return attributeValue.equals(value);
            default:
                throw new IllegalArgumentException("Invalid operator");
        }
    }

    public static boolean combineResults(String operator, boolean leftResult, boolean rightResult) {
        switch (operator) {
            case "AND":
                return leftResult && rightResult;
            case "OR":
                return leftResult || rightResult;
            default:
                throw new IllegalArgumentException("Invalid operator");
        }
    }

    public static Node createRule(String ruleString) {
        return parseToAST(ruleString);
    }

    public static Node combineRules(String[] rules) {
        Node root = null;
        for (String rule : rules) {
            Node ast = createRule(rule);
            if (root == null) {
                root = ast;
            } else {
                root = new Node("operator", root, ast, null); // Simple AND combination
            }
        }
        return root;
    }

    public static boolean evaluateRule(Node ast, Map<String, String> data) {
        if (ast.nodeType.equals("operand")) {
            String attribute = ast.value.get("attribute");
            String operator = ast.value.get("operator");
            String value = ast.value.get("value");
            return evaluateCondition(data.get(attribute), operator, value);
        } else if (ast.nodeType.equals("operator")) {
            boolean leftResult = evaluateRule(ast.left, data);
            boolean rightResult = evaluateRule(ast.right, data);
            return combineResults(ast.value.get("operator"), leftResult, rightResult);
        }
        throw new IllegalArgumentException("Invalid AST node");
    }

    // Test Cases
    public static void main(String[] args) {
        // Test create_rule
        Node rule1 = createRule("((age > 30 AND department = \"Sales\") OR (age < 25 AND department = \"Marketing\")) AND (salary > 50000 OR experience > 5)");
        assert rule1 != null : "Rule creation failed!";

        // Test combine_rules
        String[] rules = {
            "((age > 30 AND department = \"Sales\"))",
            "((age < 25 AND department = \"Marketing\"))"
        };
        Node combinedAST = combineRules(rules);
        assert combinedAST != null : "Combination of rules failed!";

        // Test evaluate_rule
        Map<String, String> data = new HashMap<>();
        data.put("age", "35");
        data.put("department", "Sales");
        data.put("salary", "60000");
        data.put("experience", "3");

        assert evaluateRule(rule1, data) : "Evaluation of rule failed!";

        data.put("age", "25");
        assert !evaluateRule(rule1, data) : "Evaluation of rule failed!";

        System.out.println("All tests passed!");
    }
}
