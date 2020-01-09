<template>
	<div class="card material-table" ref="table">
		<div class="table-header">
			<span class="table-title">{{ title }}</span>
			<div class="actions">
				<a v-for="(button, index) in customButtons" v-if="button.hide ? !button.hide : true"
					:key="index"
					href="javascript:undefined"
					class="waves-effect btn-flat nopadding"
					@click="button.onclick"
				>
					<i class="material-icons">{{ button.icon }}</i>
				</a>
				<a v-if="printable"
					href="javascript:undefined"
					class="waves-effect btn-flat nopadding"
					@click="print"
				>
					<i class="material-icons">print</i>
				</a>
				<a v-if="exportable"
					href="javascript:undefined"
					class="waves-effect btn-flat nopadding"
					@click="exportExcel"
				>
					<i class="material-icons">description</i>
				</a>
				<a v-if="searchable"
					href="javascript:undefined"
					class="waves-effect btn-flat nopadding"
					@click="search"
				>
					<i class="material-icons">search</i>
				</a>
			</div>
		</div>
		<div v-if="searching">
			<div id="search-input-container">
				<label>
					<input id="search-input" type="search" class="form-control" :placeholder="lang['search_data']"
						v-model="searchInput"
					>
				</label>
			</div>
		</div>
		<table ref="table">
			<thead>
				<tr>
					<th v-for="(column, index) in columns"
						:key="index"
						:class="(sortable ? 'sorting ' : '')
							+ (sortColumn === index ?
								(sortType === 'desc' ? 'sorting-desc' : 'sorting-asc')
								: '')
							+ (column.numeric ? ' numeric' : '')"
						:style="{width: column.width ? column.width : 'auto'}"
						@click="sort(index)"
					>
						{{ column.label }}
					</th>
					<slot name="thead-tr" />
				</tr>
			</thead>

			<tbody>
				<tr v-for="(row, index) in paginated"
					:key="index"
					:class="{ clickable : clickable }"
					@click="click(row)"
				>
					<td v-for="(column, columnIndex) in columns"
						:key="columnIndex"
						:class="{ numeric : column.numeric }"
					>
						<div v-if="!column.html">
							{{ collect(row, column.field) }}
						</div>
						<div v-if="column.html" v-html="collect(row, column.field)" />
					</td>
					<slot name="tbody-tr" :row="row" />
				</tr>

				<template v-if="rows.length === 0 && loadingAnimation === true">
					<tr v-for="n in (currentPerPage === -1 ? 10 : currentPerPage)"
						:key="n"
					>
						<td :colspan="columns.length">
							<tb-skeleton :height="15" theme="opacity" bg-color="#dcdbdc" shape="radius" />
						</td>
					</tr>
				</template>
			</tbody>
		</table>

		<div v-if="paginate" class="table-footer">
			<div :class="{'datatable-length': true, 'rtl': lang.__is_rtl}">
				<label>
					<span>{{ lang['rows_per_page'] }}:</span>
					<select class="browser-default" @change="onTableLength">
						<option v-for="(option, index) in perPageOptions"
							:key="index"
							:value="option"
							:selected="option == currentPerPage"
						>
							{{ option === -1 ? lang['all'] : option }}
						</option>
					</select>
				</label>
			</div>
			<div :class="{'datatable-info': true, 'rtl': lang.__is_rtl}">
				<span>{{ (currentPage - 1) * currentPerPage ? (currentPage - 1) * currentPerPage : 1 }}
					-{{ Math.min(processedRows.length, currentPerPage * currentPage) }}
				</span>
				<span>
					{{ lang['out_of_pages'] }}
				</span>
				<span>
					{{ processedRows.length }}
				</span>
			</div>
			<div>
				<ul class="material-pagination">
					<li>
						<a href="javascript:undefined"
							class="waves-effect btn-flat"
							tabindex="0"
							@click.prevent="previousPage"
						>
							<i class="material-icons">chevron_left</i>
						</a>
					</li>
					<li>
						<a href="javascript:undefined"
							class="waves-effect btn-flat"
							tabindex="0"
							@click.prevent="nextPage"
						>
							<i class="material-icons">chevron_right</i>
						</a>
					</li>
				</ul>
			</div>
		</div>
	</div>
</template>

<script>

	! function(e, t) {
    "object" == typeof exports && "object" == typeof module ? module.exports = t() : "function" == typeof define && define.amd ? define("Fuse", [], t) : "object" == typeof exports ? exports.Fuse = t() : e.Fuse = t()
}(this, function() {
    return n = {}, o.m = r = [function(e, t) {
        e.exports = function(e) {
            return Array.isArray ? Array.isArray(e) : "[object Array]" === Object.prototype.toString.call(e)
        }
    }, function(e, t, r) {
        function l(e) {
            return (l = "function" == typeof Symbol && "symbol" == typeof Symbol.iterator ? function(e) {
                return typeof e
            } : function(e) {
                return e && "function" == typeof Symbol && e.constructor === Symbol && e !== Symbol.prototype ? "symbol" : typeof e
            })(e)
        }
        function n(e, t) {
            for (var r = 0; r < t.length; r++) {
                var n = t[r];
                n.enumerable = n.enumerable || !1, n.configurable = !0, "value" in n && (n.writable = !0), Object.defineProperty(e, n.key, n)
            }
        }
        var i = r(2),
            $ = r(8),
            E = r(0),
            o = (n(J.prototype, [{
                key: "setCollection",
                value: function(e) {
                    return this.list = e
                }
            }, {
                key: "search",
                value: function(e) {
                    var t = 1 < arguments.length && void 0 !== arguments[1] ? arguments[1] : {
                        limit: !1
                    };
                    this._log('---------\nSearch pattern: "'.concat(e, '"'));
                    var r = this._prepareSearchers(e),
                        n = r.tokenSearchers,
                        o = r.fullSearcher,
                        i = this._search(n, o),
                        a = i.weights,
                        s = i.results;
                    return this._computeScore(a, s), this.options.shouldSort && this._sort(s), t.limit && "number" == typeof t.limit && (s = s.slice(0, t.limit)), this._format(s)
                }
            }, {
                key: "_prepareSearchers",
                value: function() {
                    var e = 0 < arguments.length && void 0 !== arguments[0] ? arguments[0] : "",
                        t = [];
                    if (this.options.tokenize)
                        for (var r = e.split(this.options.tokenSeparator), n = 0, o = r.length; n < o; n += 1) t.push(new i(r[n], this.options));
                    return {
                        tokenSearchers: t,
                        fullSearcher: new i(e, this.options)
                    }
                }
            }, {
                key: "_search",
                value: function() {
                    var e = 0 < arguments.length && void 0 !== arguments[0] ? arguments[0] : [],
                        t = 1 < arguments.length ? arguments[1] : void 0,
                        r = this.list,
                        n = {},
                        o = [];
                    if ("string" == typeof r[0]) {
                        for (var i = 0, a = r.length; i < a; i += 1) this._analyze({
                            key: "",
                            value: r[i],
                            record: i,
                            index: i
                        }, {
                            resultMap: n,
                            results: o,
                            tokenSearchers: e,
                            fullSearcher: t
                        });
                        return {
                            weights: null,
                            results: o
                        }
                    }
                    for (var s = {}, c = 0, h = r.length; c < h; c += 1)
                        for (var l = r[c], u = 0, f = this.options.keys.length; u < f; u += 1) {
                            var d = this.options.keys[u];
                            if ("string" != typeof d) {
                                if (s[d.name] = {
                                        weight: 1 - d.weight || 1
                                    }, d.weight <= 0 || 1 < d.weight) throw new Error("Key weight has to be > 0 and <= 1");
                                d = d.name
                            } else s[d] = {
                                weight: 1
                            };
                            this._analyze({
                                key: d,
                                value: this.options.getFn(l, d),
                                record: l,
                                index: c
                            }, {
                                resultMap: n,
                                results: o,
                                tokenSearchers: e,
                                fullSearcher: t
                            })
                        }
                    return {
                        weights: s,
                        results: o
                    }
                }
            }, {
                key: "_analyze",
                value: function(e, t) {
                    var r = e.key,
                        n = e.arrayIndex,
                        o = void 0 === n ? -1 : n,
                        i = e.value,
                        a = e.record,
                        s = e.index,
                        c = t.tokenSearchers,
                        h = void 0 === c ? [] : c,
                        l = t.fullSearcher,
                        u = void 0 === l ? [] : l,
                        f = t.resultMap,
                        d = void 0 === f ? {} : f,
                        v = t.results,
                        p = void 0 === v ? [] : v;
                    if (null != i) {
                        var g = !1,
                            y = -1,
                            m = 0;
                        if ("string" == typeof i) {
                            this._log("\nKey: ".concat("" === r ? "-" : r));
                            var k = u.search(i);
                            if (this._log('Full text: "'.concat(i, '", score: ').concat(k.score)), this.options.tokenize) {
                                for (var S = i.split(this.options.tokenSeparator), x = [], b = 0; b < h.length; b += 1) {
                                    var M = h[b];
                                    this._log('\nPattern: "'.concat(M.pattern, '"'));
                                    for (var _ = !1, L = 0; L < S.length; L += 1) {
                                        var w = S[L],
                                            A = M.search(w),
                                            C = {};
                                        A.isMatch ? (C[w] = A.score, _ = g = !0, x.push(A.score)) : (C[w] = 1, this.options.matchAllTokens || x.push(1)), this._log('Token: "'.concat(w, '", score: ').concat(C[w]))
                                    }
                                    _ && (m += 1)
                                }
                                y = x[0];
                                for (var I = x.length, O = 1; O < I; O += 1) y += x[O];
                                y /= I, this._log("Token score average:", y)
                            }
                            var j = k.score; - 1 < y && (j = (j + y) / 2), this._log("Score average:", j);
                            var P = !this.options.tokenize || !this.options.matchAllTokens || m >= h.length;
                            if (this._log("\nCheck Matches: ".concat(P)), (g || k.isMatch) && P) {
                                var F = d[s];
                                F ? F.output.push({
                                    key: r,
                                    arrayIndex: o,
                                    value: i,
                                    score: j,
                                    matchedIndices: k.matchedIndices
                                }) : (d[s] = {
                                    item: a,
                                    output: [{
                                        key: r,
                                        arrayIndex: o,
                                        value: i,
                                        score: j,
                                        matchedIndices: k.matchedIndices
                                    }]
                                }, p.push(d[s]))
                            }
                        } else if (E(i))
                            for (var T = 0, z = i.length; T < z; T += 1) this._analyze({
                                key: r,
                                arrayIndex: T,
                                value: i[T],
                                record: a,
                                index: s
                            }, {
                                resultMap: d,
                                results: p,
                                tokenSearchers: h,
                                fullSearcher: u
                            })
                    }
                }
            }, {
                key: "_computeScore",
                value: function(e, t) {
                    this._log("\n\nComputing score:\n");
                    for (var r = 0, n = t.length; r < n; r += 1) {
                        for (var o = t[r].output, i = o.length, a = 1, s = 1, c = 0; c < i; c += 1) {
                            var h = e ? e[o[c].key].weight : 1,
                                l = (1 === h ? o[c].score : o[c].score || .001) * h;
                            1 !== h ? s = Math.min(s, l) : a *= o[c].nScore = l
                        }
                        t[r].score = 1 === s ? a : s, this._log(t[r])
                    }
                }
            }, {
                key: "_sort",
                value: function(e) {
                    this._log("\n\nSorting...."), e.sort(this.options.sortFn)
                }
            }, {
                key: "_format",
                value: function(e) {
                    var t = [];
                    if (this.options.verbose) {
                        var r = [];
                        this._log("\n\nOutput:\n\n", JSON.stringify(e, function(e, t) {
                            if ("object" === l(t) && null !== t) {
                                if (-1 !== r.indexOf(t)) return;
                                r.push(t)
                            }
                            return t
                        })), r = null
                    }
                    var n = [];
                    this.options.includeMatches && n.push(function(e, t) {
                        var r = e.output;
                        t.matches = [];
                        for (var n = 0, o = r.length; n < o; n += 1) {
                            var i = r[n];
                            if (0 !== i.matchedIndices.length) {
                                var a = {
                                    indices: i.matchedIndices,
                                    value: i.value
                                };
                                i.key && (a.key = i.key), i.hasOwnProperty("arrayIndex") && -1 < i.arrayIndex && (a.arrayIndex = i.arrayIndex), t.matches.push(a)
                            }
                        }
                    }), this.options.includeScore && n.push(function(e, t) {
                        t.score = e.score
                    });
                    for (var o = 0, i = e.length; o < i; o += 1) {
                        var a = e[o];
                        if (this.options.id && (a.item = this.options.getFn(a.item, this.options.id)[0]), n.length) {
                            for (var s = {
                                    item: a.item
                                }, c = 0, h = n.length; c < h; c += 1) n[c](a, s);
                            t.push(s)
                        } else t.push(a.item)
                    }
                    return t
                }
            }, {
                key: "_log",
                value: function() {
                    var e;
                    this.options.verbose && (e = console).log.apply(e, arguments)
                }
            }]), J);
        function J(e, t) {
            var r = t.location,
                n = void 0 === r ? 0 : r,
                o = t.distance,
                i = void 0 === o ? 100 : o,
                a = t.threshold,
                s = void 0 === a ? .6 : a,
                c = t.maxPatternLength,
                h = void 0 === c ? 32 : c,
                l = t.caseSensitive,
                u = void 0 !== l && l,
                f = t.tokenSeparator,
                d = void 0 === f ? / +/g : f,
                v = t.findAllMatches,
                p = void 0 !== v && v,
                g = t.minMatchCharLength,
                y = void 0 === g ? 1 : g,
                m = t.id,
                k = void 0 === m ? null : m,
                S = t.keys,
                x = void 0 === S ? [] : S,
                b = t.shouldSort,
                M = void 0 === b || b,
                _ = t.getFn,
                L = void 0 === _ ? $ : _,
                w = t.sortFn,
                A = void 0 === w ? function(e, t) {
                    return e.score - t.score
                } : w,
                C = t.tokenize,
                I = void 0 !== C && C,
                O = t.matchAllTokens,
                j = void 0 !== O && O,
                P = t.includeMatches,
                F = void 0 !== P && P,
                T = t.includeScore,
                z = void 0 !== T && T,
                E = t.verbose,
                K = void 0 !== E && E;
            ! function(e) {
                if (!(e instanceof J)) throw new TypeError("Cannot call a class as a function")
            }(this), this.options = {
                location: n,
                distance: i,
                threshold: s,
                maxPatternLength: h,
                isCaseSensitive: u,
                tokenSeparator: d,
                findAllMatches: p,
                minMatchCharLength: y,
                id: k,
                keys: x,
                includeMatches: F,
                includeScore: z,
                shouldSort: M,
                getFn: L,
                sortFn: A,
                verbose: K,
                tokenize: I,
                matchAllTokens: j
            }, this.setCollection(e)
        }
        e.exports = o
    }, function(e, t, r) {
        function n(e, t) {
            for (var r = 0; r < t.length; r++) {
                var n = t[r];
                n.enumerable = n.enumerable || !1, n.configurable = !0, "value" in n && (n.writable = !0), Object.defineProperty(e, n.key, n)
            }
        }
        var l = r(3),
            u = r(4),
            m = r(7),
            o = (n(k.prototype, [{
                key: "search",
                value: function(e) {
                    if (this.options.isCaseSensitive || (e = e.toLowerCase()), this.pattern === e) return {
                        isMatch: !0,
                        score: 0,
                        matchedIndices: [
                            [0, e.length - 1]
                        ]
                    };
                    var t = this.options,
                        r = t.maxPatternLength,
                        n = t.tokenSeparator;
                    if (this.pattern.length > r) return l(e, this.pattern, n);
                    var o = this.options,
                        i = o.location,
                        a = o.distance,
                        s = o.threshold,
                        c = o.findAllMatches,
                        h = o.minMatchCharLength;
                    return u(e, this.pattern, this.patternAlphabet, {
                        location: i,
                        distance: a,
                        threshold: s,
                        findAllMatches: c,
                        minMatchCharLength: h
                    })
                }
            }]), k);
        function k(e, t) {
            var r = t.location,
                n = void 0 === r ? 0 : r,
                o = t.distance,
                i = void 0 === o ? 100 : o,
                a = t.threshold,
                s = void 0 === a ? .6 : a,
                c = t.maxPatternLength,
                h = void 0 === c ? 32 : c,
                l = t.isCaseSensitive,
                u = void 0 !== l && l,
                f = t.tokenSeparator,
                d = void 0 === f ? / +/g : f,
                v = t.findAllMatches,
                p = void 0 !== v && v,
                g = t.minMatchCharLength,
                y = void 0 === g ? 1 : g;
            ! function(e) {
                if (!(e instanceof k)) throw new TypeError("Cannot call a class as a function")
            }(this), this.options = {
                location: n,
                distance: i,
                threshold: s,
                maxPatternLength: h,
                isCaseSensitive: u,
                tokenSeparator: d,
                findAllMatches: p,
                minMatchCharLength: y
            }, this.pattern = this.options.isCaseSensitive ? e : e.toLowerCase(), this.pattern.length <= h && (this.patternAlphabet = m(this.pattern))
        }
        e.exports = o
    }, function(e, t) {
        var l = /[\-\[\]\/\{\}\(\)\*\+\?\.\\\^\$\|]/g;
        e.exports = function(e, t) {
            var r = 2 < arguments.length && void 0 !== arguments[2] ? arguments[2] : / +/g,
                n = new RegExp(t.replace(l, "\\$&").replace(r, "|")),
                o = e.match(n),
                i = !!o,
                a = [];
            if (i)
                for (var s = 0, c = o.length; s < c; s += 1) {
                    var h = o[s];
                    a.push([e.indexOf(h), h.length - 1])
                }
            return {
                score: i ? .5 : 1,
                isMatch: i,
                matchedIndices: a
            }
        }
    }, function(e, t, r) {
        var E = r(5),
            K = r(6);
        e.exports = function(e, t, r, n) {
            for (var o = n.location, i = void 0 === o ? 0 : o, a = n.distance, s = void 0 === a ? 100 : a, c = n.threshold, h = void 0 === c ? .6 : c, l = n.findAllMatches, u = void 0 !== l && l, f = n.minMatchCharLength, d = void 0 === f ? 1 : f, v = i, p = e.length, g = h, y = e.indexOf(t, v), m = t.length, k = [], S = 0; S < p; S += 1) k[S] = 0;
            if (-1 !== y) {
                var x = E(t, {
                    errors: 0,
                    currentLocation: y,
                    expectedLocation: v,
                    distance: s
                });
                if (g = Math.min(x, g), -1 !== (y = e.lastIndexOf(t, v + m))) {
                    var b = E(t, {
                        errors: 0,
                        currentLocation: y,
                        expectedLocation: v,
                        distance: s
                    });
                    g = Math.min(b, g)
                }
            }
            y = -1;
            for (var M = [], _ = 1, L = m + p, w = 1 << (m <= 31 ? m - 1 : 30), A = 0; A < m; A += 1) {
                for (var C = 0, I = L; C < I;) E(t, {
                    errors: A,
                    currentLocation: v + I,
                    expectedLocation: v,
                    distance: s
                }) <= g ? C = I : L = I, I = Math.floor((L - C) / 2 + C);
                L = I;
                var O = Math.max(1, v - I + 1),
                    j = u ? p : Math.min(v + I, p) + m,
                    P = Array(j + 2);
                P[j + 1] = (1 << A) - 1;
                for (var F = j; O <= F; F -= 1) {
                    var T = F - 1,
                        z = r[e.charAt(T)];
                    if (z && (k[T] = 1), P[F] = (P[F + 1] << 1 | 1) & z, 0 !== A && (P[F] |= (M[F + 1] | M[F]) << 1 | 1 | M[F + 1]), P[F] & w && (_ = E(t, {
                            errors: A,
                            currentLocation: T,
                            expectedLocation: v,
                            distance: s
                        })) <= g) {
                        if (g = _, (y = T) <= v) break;
                        O = Math.max(1, 2 * v - y)
                    }
                }
                if (E(t, {
                        errors: A + 1,
                        currentLocation: v,
                        expectedLocation: v,
                        distance: s
                    }) > g) break;
                M = P
            }
            return {
                isMatch: 0 <= y,
                score: 0 === _ ? .001 : _,
                matchedIndices: K(k, d)
            }
        }
    }, function(e, t) {
        e.exports = function(e, t) {
            var r = t.errors,
                n = void 0 === r ? 0 : r,
                o = t.currentLocation,
                i = void 0 === o ? 0 : o,
                a = t.expectedLocation,
                s = void 0 === a ? 0 : a,
                c = t.distance,
                h = void 0 === c ? 100 : c,
                l = n / e.length,
                u = Math.abs(s - i);
            return h ? l + u / h : u ? 1 : l
        }
    }, function(e, t) {
        e.exports = function() {
            for (var e = 0 < arguments.length && void 0 !== arguments[0] ? arguments[0] : [], t = 1 < arguments.length && void 0 !== arguments[1] ? arguments[1] : 1, r = [], n = -1, o = -1, i = 0, a = e.length; i < a; i += 1) {
                var s = e[i];
                s && -1 === n ? n = i : s || -1 === n || ((o = i - 1) - n + 1 >= t && r.push([n, o]), n = -1)
            }
            return e[i - 1] && t <= i - n && r.push([n, i - 1]), r
        }
    }, function(e, t) {
        e.exports = function(e) {
            for (var t = {}, r = e.length, n = 0; n < r; n += 1) t[e.charAt(n)] = 0;
            for (var o = 0; o < r; o += 1) t[e.charAt(o)] |= 1 << r - o - 1;
            return t
        }
    }, function(e, t, r) {
        var l = r(0);
        e.exports = function(e, t) {
            return function e(t, r, n) {
                if (r) {
                    var o = r.indexOf("."),
                        i = r,
                        a = null; - 1 !== o && (i = r.slice(0, o), a = r.slice(o + 1));
                    var s = t[i];
                    if (null != s)
                        if (a || "string" != typeof s && "number" != typeof s)
                            if (l(s))
                                for (var c = 0, h = s.length; c < h; c += 1) e(s[c], a, n);
                            else a && e(s, a, n);
                    else n.push(s.toString())
                } else n.push(t);
                return n
            }(e, t, [])
        }
    }], o.c = n, o.d = function(e, t, r) {
        o.o(e, t) || Object.defineProperty(e, t, {
            enumerable: !0,
            get: r
        })
    }, o.r = function(e) {
        "undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
            value: "Module"
        }), Object.defineProperty(e, "__esModule", {
            value: !0
        })
    }, o.t = function(t, e) {
        if (1 & e && (t = o(t)), 8 & e) return t;
        if (4 & e && "object" == typeof t && t && t.__esModule) return t;
        var r = Object.create(null);
        if (o.r(r), Object.defineProperty(r, "default", {
                enumerable: !0,
                value: t
            }), 2 & e && "string" != typeof t)
            for (var n in t) o.d(r, n, function(e) {
                return t[e]
            }.bind(null, n));
        return r
    }, o.n = function(e) {
        var t = e && e.__esModule ? function() {
            return e.default
        } : function() {
            return e
        };
        return o.d(t, "a", t), t
    }, o.o = function(e, t) {
        return Object.prototype.hasOwnProperty.call(e, t)
    }, o.p = "", o(o.s = 1);
    function o(e) {
        if (n[e]) return n[e].exports;
        var t = n[e] = {
            i: e,
            l: !1,
            exports: {}
        };
        return r[e].call(t.exports, t, t.exports, o), t.l = !0, t.exports
    }
    var r, n
});

	export default {
		props: {
			title: {
				type: String,
				required: true,
			},

			columns: {
				type: Array,
				required: true,
			},

			rows: {
				type: Array,
				required: true,
			},

			clickable: {
				type: Boolean,
				required: false,
				default: true,
			},

			customButtons: {
				type: Function,
				required: false,
				default: () => [],
			},

			perPage: {
				type: Array,
				required: false,
				default: () => [10, 20, 30, 40, 50],
			},

			defaultPerPage: {
				type: Number,
				required: false,
				default: null,
			},

			sortable: {
				type: Boolean,
				required: false,
				default: true,
			},

			searchable: {
				type: Boolean,
				required: false,
				default: true,
			},

			exactSearch: {
				type: Boolean,
				required: false,
				default: false,
			},

			serverSearch: {
				type: Boolean,
				required: false,
				default: false,
			},

			serverSearchFunc: {
				type: Function,
				required: false,
				default: () => {},
			},

			paginate: {
				type: Boolean,
				required: false,
				default: true,
			},

			exportable: {
				type: Boolean,
				required: false,
				default: true,
			},

			printable: {
				type: Boolean,
				required: false,
				default: true,
			},

			locale: {
				type: String,
				required: false,
				default: 'en',
			},

			initSortCol: {
				type: Number,
				required: false,
				default: -1,
			},

			loadingAnimation: {
				type: Boolean,
				required: false,
				default: true,
			},

		},

		data: () => ({
			currentPage: 1,
			currentPerPage: 10,
			sortColumn: -1,
			sortType: 'asc',
			searching: false,
			searchInput: '',
		}),

		methods: {
			nextPage() {
				if (this.processedRows.length > this.currentPerPage * this.currentPage)
					++this.currentPage;
			},

			previousPage() {
				if (this.currentPage > 1)
					--this.currentPage;
			},

			onTableLength(e) {
				this.currentPerPage = parseInt(e.target.value);
			},

			sort(index) {
				if (!this.sortable)
					return;
				if (this.sortColumn === index) {
					this.sortType = this.sortType === 'asc' ? 'desc' : 'asc';
				} else {
					this.sortType = 'asc';
					this.sortColumn = index;
				}
			},

			search(e) {
				this.searching = !this.searching;
			},

			click(row) {
				if(!this.clickable){
					return;
				}

				if(getSelection().toString()){
					// Return if some text is selected instead of firing the row-click event.
					return;
				}

				this.$emit('row-click', row);
			},

			exportExcel() {
				const mimeType = 'data:application/vnd.ms-excel';
				const html = this.renderTable().replace(/ /g, '%20');

				// eslint-disable-next-line eqeqeq
				const documentPrefix = this.title != '' ? this.title.replace(/ /g, '-') : 'Sheet';
				const d = new Date();

				const dummy = document.createElement('a');
				dummy.href = mimeType + ', ' + html;
				dummy.download = documentPrefix
					+ '-' + d.getFullYear() + '-' + (d.getMonth() + 1) + '-' + d.getDate()
					+ '-' + d.getHours() + '-' + d.getMinutes() + '-' + d.getSeconds()
					+ '.xls';
				document.body.appendChild(dummy);
				dummy.click();
			},

			print() {
				const clonedTable = this.$refs.table.cloneNode(true);
				this.synchronizeCssStyles(this.$refs.table, clonedTable, true);

				clonedTable.style.maxWidth = '100%';
				clonedTable.style.boxShadow = '0px 0px 0px 1px rgba(0,0,0,0.2)';
				clonedTable.style.margin = '8px auto';
				clonedTable.querySelector('.actions').remove();
				clonedTable.querySelector('.material-pagination').remove();
				clonedTable.querySelector('.datatable-length').remove();

				clonedTable.querySelectorAll('button').forEach(n => n.remove());

				let win = window.open('', '');


				win.document.body.style.fontFamily = '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen-Sans, Ubuntu, Cantarell, "Helvetica Neue", sans-serif';
				win.document.body.innerHTML = (clonedTable.outerHTML);

				win.print();

				win.close();
			},

			renderTable() {
				let table = '<table><thead>';

				table += '<tr>';
				for (let i = 0; i < this.columns.length; i++) {
					const column = this.columns[i];
					table += '<th>';
					table += 	column.label;
					table += '</th>';
				}
				table += '</tr>';

				table += '</thead><tbody>';

				for (let i = 0; i < this.rows.length; i++) {
					const row = this.rows[i];
					table += '<tr>';
					for (let j = 0; j < this.columns.length; j++) {
						const column = this.columns[j];
						table += '<td>';
						table +=	this.collect(row, column.field);
						table += '</td>';
					}
					table += '</tr>';
				}

				table += '</tbody></table>';

				return table;
			},

			dig(obj, selector) {
				let result = obj;
				const splitter = selector.split('.');

				for (let i = 0; i < splitter.length; i++) {
					if (result == undefined)
						return undefined;

					result = result[splitter[i]];
				}

				return result;
			},

			collect(obj, field) {
				if (typeof(field) === 'function')
					return field(obj);
				else if (typeof(field) === 'string')
					return this.dig(obj, field);
				else
					return undefined;
			},

			/* https://codebottle.io/s/31b70f5391
			 *
			 * @author: Luigi D'Amico (www.8bitplatoon.com)
			 */
			synchronizeCssStyles(src, destination, recursively) {
				destination.style.cssText = this.getComputedStyleCssText(src);

				if (recursively) {
					const vSrcElements = src.getElementsByTagName('*');
					const vDstElements = destination.getElementsByTagName('*');

					for (var i = vSrcElements.length; i--;) {
						const vSrcElement = vSrcElements[i];
						const vDstElement = vDstElements[i];
						vDstElement.style.cssText = this.getComputedStyleCssText(vSrcElement);
					}
				}
			},

			// https://gist.github.com/johnkpaul/1754808
			//
			// Please delete Firefox.
			getComputedStyleCssText(element) {
				const cssObject = window.getComputedStyle(element);
				const cssAccumulator = [];

				if (cssObject.cssText !== ''){
					return cssObject.cssText;
				}

				for (let prop in cssObject){
					if (typeof cssObject[prop] === 'string'){
						cssAccumulator.push(prop + ' : ' + cssObject[prop]);
					}
				}

				return cssAccumulator.join('; ');
			},
		},

		watch: {
			perPageOptions(newOptions, oldOptions) {
				// If defaultPerPage is provided and it's a valid option, set as current per page
				if (newOptions.indexOf(this.defaultPerPage) > -1) {
					this.currentPerPage = parseInt(this.defaultPerPage);
				} else {
					// Set current page to first value
					this.currentPerPage = newOptions[0];
				}
			},

			searchInput(newSearchInput) {
				if (this.searching && this.serverSearch && this.serverSearchFunc)
					this.serverSearchFunc(newSearchInput);
			},
		},

		computed: {
			perPageOptions() {
				let options = (Array.isArray(this.perPage) && this.perPage) || [10, 20, 30, 40, 50];

				// Force numbers
				options = options.map(v => parseInt(v));


				// Sort options
				options.sort((a,b) => a - b);

				// And add "All"
				options.push(-1);


				return options;
			},

			processedRows() {
				let computedRows = this.rows;

				if (this.sortable !== false)
					computedRows = computedRows.sort((x,y) => {
						if (!this.columns[this.sortColumn])
							return 0;

						const cook = x => {
							x = this.collect(x, this.columns[this.sortColumn].field);
							if (typeof(x) === 'string') {
								x = x.toLowerCase();
								if (this.columns[this.sortColumn].numeric)
									x = x.indexOf('.') >= 0 ? parseFloat(x) : parseInt(x);
							}
							return x;
						};

						x = cook(x);
						y = cook(y);

						return (x < y ? -1 : (x > y ? 1 : 0)) * (this.sortType === 'desc' ? -1 : 1);
					});

				if (this.searching && !this.serverSearch && this.searchInput) {
					const searchConfig = { keys: this.columns.map(c => c.field) };

					// Enable searching of numbers (non-string)
					// Temporary fix of https://github.com/krisk/Fuse/issues/144
					searchConfig.getFn = (obj, path) => {
						const property = this.dig(obj, path);
						if(Number.isInteger(property))
							return JSON.stringify(property);
						return property;
					};

					if (this.exactSearch) {
						//return only exact matches
						searchConfig.threshold = 0,
						searchConfig.distance = 0;
					}

					computedRows = (new Fuse(computedRows, searchConfig)).search(this.searchInput);
				}

				return computedRows;
			},

			paginated() {
				let paginatedRows = this.processedRows;

				if (this.paginate && this.currentPerPage !== -1)
					paginatedRows = paginatedRows.slice(
						(this.currentPage - 1) * this.currentPerPage,
						this.currentPerPage === -1 ? paginatedRows.length + 1 : this.currentPage * this.currentPerPage
					);

				return paginatedRows;
			},

			lang() {
				return this.locale in locales ? locales[this.locale] : locales['en'];
			},
		},

		mounted() {
			if (!(this.locale in locales))
				console.error(`vue-materialize-datable: Invalid locale '${this.locale}'`);
			this.sortColumn = this.initSortCol;
		},
	};
</script>

<style scoped>
	div.material-table {
		padding: 0;
	}

	tr.clickable {
		cursor: pointer;
	}

	#search-input {
		margin: 0;
		border: transparent 0 !important;
		height: 48px;
		color: rgba(0, 0, 0, .84);
	}

	#search-input-container {
		padding: 0 14px 0 24px;
		border-bottom: solid 1px #DDDDDD;
	}

	table {
		table-layout: fixed;
	}

	.table-header {
		height: 64px;
		padding-left: 24px;
		padding-right: 14px;
		-webkit-align-items: center;
		-ms-flex-align: center;
		align-items: center;
		display: flex;
		-webkit-display: flex;
		border-bottom: solid 1px #DDDDDD;
	}

	.table-header .actions {
		display: -webkit-flex;
		margin-left: auto;
	}

	.table-header .btn-flat {
			min-width: 36px;
			padding: 0 8px;
	}

	.table-header input {
		margin: 0;
		height: auto;
	}

	.table-header i {
		color: rgba(0, 0, 0, 0.54);
		font-size: 24px;
	}

	.table-footer {
		height: 56px;
		padding-left: 24px;
		padding-right: 14px;
		display: -webkit-flex;
		display: flex;
		-webkit-flex-direction: row;
		flex-direction: row;
		-webkit-justify-content: flex-end;
		justify-content: flex-end;
		-webkit-align-items: center;
		align-items: center;
		font-size: 12px !important;
		color: rgba(0, 0, 0, 0.54);
	}

	.table-footer .datatable-length {
		display: -webkit-flex;
		display: flex;
	}

	.table-footer .datatable-length select {
		outline: none;
	}

	.table-footer label {
		font-size: 12px;
		color: rgba(0, 0, 0, 0.54);
		display: -webkit-flex;
		display: flex;
		-webkit-flex-direction: row;
		/* works with row or column */

		flex-direction: row;
		-webkit-align-items: center;
		align-items: center;
		-webkit-justify-content: center;
		justify-content: center;
	}

	.table-footer .select-wrapper {
		display: -webkit-flex;
		display: flex;
		-webkit-flex-direction: row;
		/* works with row or column */

		flex-direction: row;
		-webkit-align-items: center;
		align-items: center;
		-webkit-justify-content: center;
		justify-content: center;
	}

	.table-footer .datatable-info,
	.table-footer .datatable-length {
		margin-right: 32px;
	}

	.table-footer .material-pagination {
		display: flex;
		-webkit-display: flex;
		margin: 0;
	}

	.table-footer .material-pagination li a {
		color: rgba(0, 0, 0, 0.54);
		padding: 0 8px;
		font-size: 24px;
	}

	.table-footer .select-wrapper input.select-dropdown {
		margin: 0;
		border-bottom: none;
		height: auto;
		line-height: normal;
		font-size: 12px;
		width: 40px;
		text-align: right;
	}

	.table-footer select {
		background-color: transparent;
		width: auto;
		padding: 0;
		border: 0;
		border-radius: 0;
		height: auto;
		margin-left: 20px;
	}

	.table-title {
		font-size: 20px;
		color: #000;
	}

	table tr td {
		padding: 0 0 0 56px;
		height: 48px;
		font-size: 13px;
		color: rgba(0, 0, 0, 0.87);
		border-bottom: solid 1px #DDDDDD;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}

	table td, table th {
		border-radius: 0;
	}

	table tr td a {
		color: inherit;
	}

	table tr td a i {
		font-size: 18px;
		color: rgba(0, 0, 0, 0.54);
	}

	table tr {
		font-size: 12px;
	}

	table th {
		font-size: 12px;
		font-weight: 500;
		color: #757575;
		cursor: pointer;
		white-space: nowrap;
		padding: 0;
		height: 56px;
		padding-left: 56px;
		vertical-align: middle;
		outline: none !important;

		overflow: hidden;
		text-overflow: ellipsis;
	}

	table th:hover {
		overflow: visible;
		text-overflow: initial;
	}

	table th.sorting-asc,
	table th.sorting-desc {
		color: rgba(0, 0, 0, 0.87);
	}

	table th.sorting:after,
	table th.sorting-asc:after {
		font-family: 'Material Icons';
		font-weight: normal;
		font-style: normal;
		font-size: 16px;
		line-height: 1;
		letter-spacing: normal;
		text-transform: none;
		display: inline-block;
		word-wrap: normal;
		-webkit-font-feature-settings: 'liga';
		-webkit-font-smoothing: antialiased;
		content: "arrow_back";
		-webkit-transform: rotate(90deg);
		display: none;
		vertical-align: middle;
	}

	table th.sorting:hover:after,
	table th.sorting-asc:after,
	table th.sorting-desc:after {
		display: inline-block;
	}

	table th.sorting-desc:after {
		content: "arrow_forward";
	}

	table tbody tr:hover {
		background-color: #EEE;
	}

	table th:last-child,
	table td:last-child {
		padding-right: 14px;
	}

	table th:first-child, table td:first-child {
		padding-left: 24px;
	}

	.rtl {
		direction: rtl;
	}
</style>
