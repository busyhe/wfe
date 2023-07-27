"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
exports.canSimplifyQuantifier = void 0;
const refa_1 = require("refa");
const regexp_ast_analysis_1 = require("regexp-ast-analysis");
const _1 = require(".");
function weakCachedFn(fn) {
    const cache = new WeakMap();
    return (value) => {
        let result = cache.get(value);
        if (result === undefined) {
            result = fn(value);
            cache.set(value, result);
        }
        return result;
    };
}
const containsAssertions = weakCachedFn((node) => {
    return (0, regexp_ast_analysis_1.hasSomeDescendant)(node, (n) => n.type === "Assertion");
});
const cachedGetPossiblyConsumedChar = weakCachedFn((flags) => {
    return weakCachedFn((element) => (0, _1.getPossiblyConsumedChar)(element, flags));
});
const CANNOT_SIMPLIFY = { canSimplify: false };
function canSimplifyQuantifier(quantifier, flags, parser) {
    if (quantifier.min === quantifier.max) {
        return CANNOT_SIMPLIFY;
    }
    if ((0, regexp_ast_analysis_1.isZeroLength)(quantifier)) {
        return CANNOT_SIMPLIFY;
    }
    if (containsAssertions(quantifier)) {
        return CANNOT_SIMPLIFY;
    }
    const direction = (0, regexp_ast_analysis_1.getMatchingDirection)(quantifier);
    const preceding = getPrecedingQuantifiers(quantifier, direction);
    if (!preceding) {
        return CANNOT_SIMPLIFY;
    }
    return canAbsorb(preceding, { direction, flags, parser, quantifier });
}
exports.canSimplifyQuantifier = canSimplifyQuantifier;
function canAbsorb(initialPreceding, options) {
    const { direction, flags, parser, quantifier } = options;
    const preceding = removeTargetQuantifier(initialPreceding, quantifier, direction);
    if (!preceding) {
        return CANNOT_SIMPLIFY;
    }
    const dependencies = [...preceding];
    const CAN_SIMPLIFY = {
        canSimplify: true,
        dependencies,
    };
    const fast = everyMaybe(preceding, (q) => canAbsorbElementFast(q, quantifier.element, flags));
    if (typeof fast === "boolean") {
        return fast ? CAN_SIMPLIFY : CANNOT_SIMPLIFY;
    }
    const formal = everyMaybe(fast, (q) => canAbsorbElementFormal(q, quantifier.element, parser));
    if (typeof formal === "boolean") {
        return formal ? CAN_SIMPLIFY : CANNOT_SIMPLIFY;
    }
    return formal.every((q) => {
        const parts = splitQuantifierIntoTails(q, direction);
        if (!parts)
            return false;
        const result = canAbsorb(parts, options);
        if (result.canSimplify)
            dependencies.push(...result.dependencies);
        return result.canSimplify;
    })
        ? CAN_SIMPLIFY
        : CANNOT_SIMPLIFY;
}
function everyMaybe(array, fn) {
    const maybe = [];
    for (const item of array) {
        const result = fn(item);
        if (result === false)
            return false;
        if (result === undefined)
            maybe.push(item);
    }
    if (maybe.length === 0)
        return true;
    return maybe;
}
function canAbsorbElementFast(quantifier, element, flags) {
    if (!quantifier.greedy) {
        return false;
    }
    if (!isNonFinite(quantifier)) {
        return false;
    }
    const qChar = cachedGetPossiblyConsumedChar(flags)(quantifier.element);
    const eChar = cachedGetPossiblyConsumedChar(flags)(element);
    if (qChar.char.isDisjointWith(eChar.char)) {
        return false;
    }
    if (eChar.exact && !eChar.char.without(qChar.char).isEmpty) {
        return false;
    }
    if (containsAssertions(quantifier) || containsAssertions(element)) {
        return undefined;
    }
    if (quantifier.element.type === "Character" ||
        quantifier.element.type === "CharacterClass" ||
        quantifier.element.type === "CharacterSet") {
        if (quantifier.max !== Infinity) {
            return false;
        }
        if (qChar.exact && qChar.char.isSupersetOf(eChar.char)) {
            return true;
        }
    }
    return undefined;
}
function isNonFinite(node) {
    return (0, regexp_ast_analysis_1.hasSomeDescendant)(node, (n) => n.type === "Quantifier" &&
        n.max === Infinity &&
        !(0, regexp_ast_analysis_1.isZeroLength)(n.element), (n) => n.type !== "Assertion");
}
function toNfa(element, parser) {
    const { expression, maxCharacter } = parser.parseElement(element, {
        maxNodes: 1000,
        assertions: "throw",
        backreferences: "throw",
    });
    return refa_1.NFA.fromRegex(expression, { maxCharacter }, {}, new refa_1.NFA.LimitedNodeFactory(1000));
}
function canAbsorbElementFormal(quantifier, element, parser) {
    if (containsAssertions(quantifier) || containsAssertions(element)) {
        return undefined;
    }
    try {
        const qNfa = toNfa(quantifier, parser);
        const qDfa = refa_1.DFA.fromFA(qNfa, new refa_1.DFA.LimitedNodeFactory(1000));
        const eNfa = toNfa(element, parser);
        eNfa.quantify(0, 1);
        qNfa.append(eNfa);
        const qeDfa = refa_1.DFA.fromFA(qNfa, new refa_1.DFA.LimitedNodeFactory(1000));
        qDfa.minimize();
        qeDfa.minimize();
        const equal = qDfa.structurallyEqual(qeDfa);
        return equal;
    }
    catch (_a) {
    }
    return undefined;
}
function splitQuantifierIntoTails(quantifier, direction) {
    if ((0, regexp_ast_analysis_1.isPotentiallyZeroLength)(quantifier)) {
        return undefined;
    }
    return getTailQuantifiers(quantifier.element, direction);
}
function removeTargetQuantifier(quantifiers, target, direction) {
    const result = [];
    for (const q of quantifiers) {
        if ((0, regexp_ast_analysis_1.hasSomeDescendant)(q, target)) {
            const inner = splitQuantifierIntoTails(q, direction);
            if (inner === undefined) {
                return undefined;
            }
            const mapped = removeTargetQuantifier(inner, target, direction);
            if (mapped === undefined) {
                return undefined;
            }
            result.push(...mapped);
        }
        else {
            result.push(q);
        }
    }
    return result;
}
function assertNever(value) {
    throw new Error(`Invalid value: ${value}`);
}
function* iterReverse(array) {
    for (let i = array.length - 1; i >= 0; i--) {
        yield array[i];
    }
}
function unionQuantifiers(sets) {
    const result = [];
    for (const set of sets) {
        if (set === undefined) {
            return undefined;
        }
        result.push(...set);
    }
    if (result.length === 0)
        return undefined;
    return [...new Set(result)];
}
function getTailQuantifiers(element, direction) {
    switch (element.type) {
        case "Assertion":
        case "Backreference":
        case "Character":
        case "CharacterClass":
        case "CharacterSet":
            return undefined;
        case "Quantifier":
            return [element];
        case "Group":
        case "CapturingGroup":
            return unionQuantifiers(element.alternatives.map((a) => getTailQuantifiers(a, direction)));
        case "Alternative": {
            const elements = direction === "ltr"
                ? iterReverse(element.elements)
                : element.elements;
            for (const e of elements) {
                if ((0, regexp_ast_analysis_1.isEmpty)(e))
                    continue;
                if (e.type === "Quantifier") {
                    return [e];
                }
                return undefined;
            }
            const { parent } = element;
            if (parent.type === "Pattern") {
                return undefined;
            }
            if (parent.type === "Assertion") {
                return undefined;
            }
            return getPrecedingQuantifiers(parent, direction);
        }
        default:
            return assertNever(element);
    }
}
function getPrecedingQuantifiers(element, direction) {
    const parent = element.parent;
    if (parent.type === "Quantifier") {
        if (parent.max === 0) {
            return undefined;
        }
        if (parent.max === 1) {
            return getPrecedingQuantifiers(parent, direction);
        }
        return unionQuantifiers([
            getPrecedingQuantifiers(parent, direction),
            getTailQuantifiers(parent.element, direction),
        ]);
    }
    if (parent.type !== "Alternative") {
        return undefined;
    }
    const inc = direction === "ltr" ? -1 : +1;
    const { elements } = parent;
    const elementIndex = elements.indexOf(element);
    for (let precedingIndex = elementIndex + inc; precedingIndex >= 0 && precedingIndex < elements.length; precedingIndex += inc) {
        const preceding = parent.elements[precedingIndex];
        if ((0, regexp_ast_analysis_1.isEmpty)(preceding))
            continue;
        return getTailQuantifiers(preceding, direction);
    }
    if (parent.parent.type === "Pattern") {
        return undefined;
    }
    return getPrecedingQuantifiers(parent.parent, direction);
}
